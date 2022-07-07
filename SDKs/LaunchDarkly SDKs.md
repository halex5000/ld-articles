# What to Expect When You're Expecting a LaunchDarkly SDK

If you're like me, when you're looking to integrate a new service into your application, you head straight for the API docs. I start researching the API calls I need to make and, since I am generally focused on JavaScript and frontend, formulating the `fetch` requests I'll need. Some tools provide SDKs that help simplify some of this logic or speed up the process of getting started, but in my experience many, if not most, of these are just API wrappers.  They may make it easier to get started but ultimately they largely just perform the same `fetch` requests I would make myself.

You might be inclined to think this is the case with LaunchDarkly's SDKs but it is incredibly far from the truth. There's a ton more going on in the SDK beyond just wrapping API calls.  In fact, I'd go so far as to say that our [25 SDKs and counting](https://docs.launchdarkly.com/sdk#available-sdks) are critical features of the tool, offering a whole additional layer of functionality on top of the product that is indispensible. In this post, I want to explore some of those features to give you a better sense of the value packed into each SDK.

## Why are there two types of LaunchDarkly SDKs?

If you've used LaunchDarkly already, you may have noticed that different types of SDKs require different types of keys. For instance, server-side SDKs require the an SDK key for the environment they are connecting to while some require a mobile key and others require a client-side ID. Why?

LaunchDarkly SDKs are divided into two different categories: client-side and server-side – mobile SDKs are also considered client-side SDKs even though they use a separate key.

### Considerations for server-side  vs. client-side SDKs

When you stop to think about the needs and security requirements around evaluating server-side flags versus client-side flags, it quickly becomes apparent how different they are.

* A server-side SDK has less concern for bandwidth than a client-side SDK. Ideally your server can pull and process large amounts of data quickly, whereas a client, whether that is a mobile app or browser, does not always have a reliable data connection and, even when it does, the speed may be limited. For instance, in many areas of the world your phone may only manage a slow 3G connection.
* Server-side and client-side SDKs have different security requirements. A server-side SDK has fewer limitations in locally caching potentially sensitive data like full flag and user targeting data. This isn't data that can be sniffed, but would require full access to your server. Meanwhile, it is relatively easy to see the data coming into a client-side application simply by monitoring http requests.

### Ok then, why the mobile key?

While ultimately they're still considered client-side apps, mobile apps have some additional considerations. First, it is a relatively common requirement for a mobile app to be able to connect to multiple LaunchDarkly environments at once, for example different environments for Android and iOS.  This is something our mobile SDKs support. In addition, it is also common for mobile apps to potentially lose a data connection, so our mobile SDKs continuously monitor the connection state to LaunchDarkly.

> For a more detailed discussion of client-side versus server-side SDKs, [check our documentation](https://docs.launchdarkly.com/sdk/concepts/client-side-server-side).

## What do I want? Incredibly fast flag evaluation

Once you start using feature flags in your codebase, it usually doesn't take long before you find you're using flags everywhere. Obviously, if your code has lots of feature flags, you need them to be extremely fast and completely reliable. LaunchDarkly's SDKs include a number of features that make flag evaluation extremely fast.

### Server-side SDKs

On the server-side, LaunchDarkly SDKs use a combination or streaming (or polling) of data and in-memory caching to make flag evaluations immediate. Here's how that works:

1. When the SDK client is initialized, it opens a streaming connection to LaunchDarkly (this can be overridden to use polling, but streaming is more efficient and recommended whenever possible).
2. LaunchDarkly sends over the full details of flag rules and segments, which are kept in an in-memory cache or configured external persistent data store like Redis or DynamoDB, for example.
3. Updates to flag rules or segments are streamed to the cache.
4. The SDK evaluates flag vations against flag rule data from the cache or data store without needing to communicate directly to LaunchDarkly.

What this means is that the SDK can reply to flag evaluation requests with almost no latency. This is because it can retrieve flag rules and user targeting without ever needing to communicate with LaunchDarkly because the data is pulled from the cache and evaluated locally within the SDK. This also means that the SDK can even work in an offline mode if it is temporarily unable to get data from LaunchDarkly.

> Note that streaming is not available in the Apex and PHP SDKs.

### Client-side SDKs

Because of the bandwidth and security considerations required for client-side SDKs, they handle things a bit differently.

1. The SDK client is initialized with the user data (because each client is a unique user).
2. LaunchDarkly performs the flag evaluation and sends over flag results for the user and caches them locally. How the data is cached depends on the platform. For example, LocalStorage is used to cache flag values in the browser.
3. Due to data and bandwidth considerations, the SDK does not automatically stream updates to the client. However, subscribing the the `change` event for flags or a particular flag will open a streaming connection for real-time updates.
4. The SDK gets flag variations from the cache or via stream updates.

The combination of local caching and streaming of updates when requested makes flag evaluation on the client as fast as possible, while keeping full flag data and user segments on LaunchDarkly rather than passing them to the client keeps any sensitive data secure while preserving limited bandwidth.

## Flag evaluation...more complicated than you'd think

There's a lot more to flag evaluation than you'd think. Things like user targeting, progressive rollouts and experimentation, just to name a few, make the logic behind getting a flag value fairly complex. Luckily our server-side SDKs come with the full feature flag evaluation algorithm built in. This is what allows your server-side application to evaluate potentially thousands of user connections almost instantaneously because the the SDK doesn't need to talk to LaunchDarkly and the flag evaluation logic is built-in.

## But wait, there's more...

It's not enough that we just evaluate flags. One of the best parts of LaunchDarkly is the data it can provide on which features are being used and how they are performing. But in order to do that we need to record analytics events for every flag evaluation and send those back in the most efficient manner possible.

Analytics data is critical to many of your favorite features in LaunchDarkly including the user dashboard, targeting rules, debugger and experimentation. In order to make these work, LaunchDarkly's SDKs handle a long list of different types of analytics event data, including your own custom events should you choose to use them.

The good news is that you don't really need to think about any of this for it to work. All of the analytics data is handled automatically via the event processing that is built into our SDKs and done in a manner so as to reduce load and limit bandwidth usage.

Rather than send constant network requests for events, the SDK manages an event buffer. This event buffer is flushed on a regular basis whereby all pending analytics data is sent ot LaunchDarkly in bulk. This flush interval is configurable but is typically a few seconds on the server-side to about 30 seconds on the client-side. The net result is that you get the valuable analytics data built-in without needing to worry about how it might impact your application load.

> For more details on the types of events LaunchDarkly's SDKs send, [check the documentation](https://docs.launchdarkly.com/sdk/concepts/events).

## Go configure

If you're going to use feature flags, you'll definitely want them to play nicely with your existing infrastructure. And if they're gonna play nicely with your existing infrastructure, you definitely don't want to have to spend a ton of needless time and effort getting it to work. That's why LaunchDarkly's SDKs work hard to make additional configurations easy to set up.

For example, let's say you have a lot of feature flags and want to reduce the time it takes to get your app up and running after a restart. Rather than using the default in-memory cache, you might want to configure LaunchDarkly to use a persistent data store that is part of your existing infrastructure like Redis, DynamoDB or Consul. In this case, LaunchDarkly can be initialized using values from the data store without waiting on the LaunchDarkly servers. Doing this can literally take just a handful of lines of code. For example, here's Node.js:

```javascript
const ld = require('launchdarkly-node-server-sdk');

const store = SomeKindOfFeatureStore(storeOptions);
const options = {
  featureStore: store
};
const client = ld.init('YOUR_SDK_KEY', config);
```

If you're on AWS, the ease of being able to configure a persistent data store like DynamoDB is a huge benefit when accessing flags from within a Lambda function, for instance.

> To learn more about how to use persistent data stores, [check our documentation](https://docs.launchdarkly.com/sdk/features/storing-data).

In addition, LaunchDarkly provides something called the [Relay Proxy](https://docs.launchdarkly.com/home/relay-proxy), which is designed to help reduce the number of outbound network connections while providing a number of other potential benefits. While this is a tool useful for [very specific but not uncommon scenarios](https://docs.launchdarkly.com/home/relay-proxy/?q=relay#determining-if-your-configuration-is-a-good-use-case-for-the-relay-proxy), it's nice that it only takes changing a few SDK configuration settings to get it working in your code using the SDKs once the Relay Proxy is set up within your network.

## When you need the API

Hopefully I've convinced you that you want to use the SDK because the SDK is awesome. Of course, there are still situations where the SDK isn't enough and you need the API. The SDK is very focused on getting flag variations, while the API gives you access to just about anything you can do in the LaunchDarkly dashboard. So if you are creating automations or pulling data into custom dashboards, for example, you'll want to go with the API. But otherwise, it's SDK all the way. 😄