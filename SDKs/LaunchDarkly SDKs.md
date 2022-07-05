# What to Expect When You're Expecting a LaunchDarkly SDK

If you're like me, when you're looking to integrate a new service into your application, you head straight for the API docs. I start researching the API calls I need to make and, since I am generally focused on JavaScript and frontend, formulating the `fetch` requests my client will need. Some tools provide SDKs that help simplify some of this logic or speed up the process of getting starte, but are ultimately just API wrappers.

This is not the case with LaunchDarkly. There's a ton more going on in the SDK beyond just wrapping API calls for you in a convenient manner.  In fact, I'd go so far as to say that our [25 SDKs and counting](https://docs.launchdarkly.com/sdk#available-sdks) are critical features of the tool, offering a whole additional layer of functionality on top of the product that ends up being indispensible. In this post, I want to explore some of those features to give you a better sense of the value packed into each SDK.

## The Two Types of SDKs

Before I can dig into some of the benefits of using the SDKs, I first need to explain the two basic categories of SDKs and why each has different security and bandwidth needs.

### Server-side SDKs and Client-side SDKs

If you start to think about the needs and security requirements around evaluating server-side flags versus client-side flags, it quickly becomes apparent how different their requirements are.

* A server-side SDK has less concern for bandwidth than a client-side SDK. Ideally your server can pull and process large amounts of data quickly, whereas a client, whether that is a mobile app or browser, does not always have a reliable data connection and, even when it does, the speed may be limited. For instance, in many areas of the world your phone may only manage a slow 3G connection.
* Server-side and client-side SDKs have different security requirements. A server-side SDK has fewer limitations in locally caching potentially sensitive data like full flag and user targeting data. This isn't data that can be sniffed, but would require full access to your server. Meanwhile, it is relatively easy to see the data coming into a client-side application simply by monitoring http requests.

For these reasons, we have two broad categories of SDKS, client-side and server-side SDKs, and each take different approaches, as we'll see below.

## Fast and Secure Flag Evaluation

As many of you probably already know, once you and your team start using feature flags in your codebase, you quickly recognize how useful they can be and start flagging everything. Obviously, if your code has lots of feature flags, you need them to be extremely fast and completely reliable. LaunchDarkly's SDKs help ensure flags meet that high bar of speed and reliability.

### Server-side SDKs

On the server-side, LaunchDarkly SDKs use a combination or streaming (or polling) of data and in-memory caching to make flag evaluations extremely fast. Here's how that works:

1. When the SDK client is initialized, it opens a streaming connection to LaunchDarkly (the streaming default can be overridden to use polling, but streaming is more efficient and recommended whenever possible).
2. LaunchDarkly sends over the full details of flag rules and segments, which are kept in an in-memory or configured external persistent data store like Redis, for example.
3. Any updates to flag rules or segments are streamed to the cache.
4. The SDK gets flag rule data directly from the cache or data store without needing to communicate directly to LaunchDarkly. Each flag variation request passes the user data which is used to evaluate the flag against the data in the cache or data store.

What this means is that the SDK can reply to flag evaluation requests with almost no latency. This is because it can retrieve flag rules and user targeting without ever needing to communicate with LaunchDarkly because the data is pulled from the cache and evaluated locally within the SDK. This also means that the SDK can even work in an offline mode if it is temporarily unable to get data from LaunchDarkly. In these cases, if the cache is populated, it can evaluate flags from the cache. Even in cases where the cache isn't populated, the SDKs allow developers to provide a flag default to prevent any issues.

> Note that streaming is not available in the Apex and PHP SDKs.

### Client-side SDKs

Because of the bandwidth and security considerations required for client-side SDKs, they handle things a bit differently.

1. The SDK client is initialized with the user data (because each client is a unique user).
2. LaunchDarkly performs the flag evaluation and sends over flag results for the user and caches them locally. For example, LocalStorage is used to cache flag values in the browser.
3. Due to data and bandwidth considerations, the SDK does not automatically stream updates to the client. However, subscribing the the `change` event for flags or a particular flag will open a streaming connection for real-time updates.
4. The SDK gets flag variations from the cache or via stream updates.

The combination of local caching and streaming of updates when requested makes flag evaluation on the client as fast as possible, while keeping full flag data and user segments on LaunchDarkly rather than passing them to the client keeps any sensitive data secure while preserving limited bandwidth.

---

streaming, local storage, in memory cache

server side
streaming in every SK except Apex and PHP. Polling in every SDK except Apex.
offline mode (?) in all but Apex

Every server-side SDK must maintain a data store that holds the last known SDK data—either an in-memory data store, or an external persistent data store (such as Redis). Data includes both feature flags and user segments. Get and GetAll (allFlagsState) operations are first performed against the data store

data stores are connecting to streaming and/or polling source for continuous updates of data.

client side SDKs do not store flags and segments, only the evaluation results, in the data store.

## Flag Evaluation Built In

As discussed above, feature flag evaluation on the server-side is actually performed by the SDK. This means that each server-side SDK comes with a version of our full feature flag evaluation algorithm. Let's examine why this is an important feature.

Unlike a client-side application in a browser or mobile app, which has a single user by definition, a single server-side application instance might be serving thousands of user connections. Imagine if each user connection had to go to LaunchDarkly to retrieve and evaluate a flag variation for every use of a flag within the request. This would quickly become a bottleneck in your application.

Instead, LaunchDarkly embeds the flag evaluation system into the SDK so that flag evaluation logic can be performed locally within the application, including user targeting for all possible users, without ever needing to contact LaunchDarkly to get the result.

## Event Recording

One of the best parts of LaunchDarkly is the data it can provide on which features are being used and how they are performing. Analytics data is critical to the funcitoning of most of LaunchDarkly's most powerful features including the user dashbaord, targeting rules, debugger and experimentation. In fact, LaunchDarkly's SDKs handle a long list of different types of analytics event data, including your own custom events should you choose to use them.

The good news is that you don't really need to think about any of this for it to work. All of the analytics data is handled automatically via the event processing that is built into our SDKs and done in a manner so as to reduce load and limit bandwidth usage. It is also worth noting that the JavaScript SDKs also properly handle when an end-user has Do Not Track enabled in their browser, wherein the SDK does not send analytics events for flag evaluations or goals.

Rather than send constant network requests for events, especially on the server side where the SDK might be handling thousands of concurrent user connections, the SDK manages an event buffer. This event buffer is flushed on a regular basis whereby all pending analytics data is sent ot LaunchDarkly in bulk. This flush interval is configurable but is typically a few seconds on the server-side to about 30 seconds on the client-side. The net result is that you get the valuable analytics data built-in without needing to worry about how it might impact your application load.

For more details on the types of events LaunchDarkly's SDKs send, [check the documentation](https://docs.launchdarkly.com/sdk/concepts/events).

## Ease of Additional Configuration

Redis, DynamoDB, Consul (??) - persistent data stores

Server side
Redis data store in all but Apex and Rust
Consul and DynamoDB in .Net, Go, Java, Node PHP Python, Ruby

LD Relay

https://docs.launchdarkly.com/sdk/features/storing-data

## When You Need the API

https://docs.launchdarkly.com/sdk/features

https://launchdarkly.atlassian.net/wiki/spaces/ENG/pages/15466516/SDK+consistency+tracker

As regards the difference between SDKs and going direct to the API, I'd say that the SDKs implement:
a carefully designed caching and streaming system that ensures most flags are evaluated instantaneously in a non-blocking manner
event recording and buffering to track how flags are used
(on the server-side) a full flag evaluation system so that targeting and other rules can be evaluated for all possible users
integration with local persistent stores for caching and offline usage (on mobile this happens automatically)
simplified configuration for LD Relay and other proxies


https://launchdarkly.atlassian.net/wiki/spaces/ENG/pages/15466516/SDK+consistency+tracker