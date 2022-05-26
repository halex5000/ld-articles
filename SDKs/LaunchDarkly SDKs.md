# What to Expect When You're Expecting a LaunchDarkly SDK

If you're like me, when you're looking to integrate a new service into your application, you head straight for the API docs. I start researching the API calls I need to make and, since I am generally focused on JavaScript and frontend, formulating the `fetch` requests my client will need. Some tools provide SDKs that help simplify some of this logic or speed up the process of getting starte, but are ultimately just API wrappers.

This is not the case with LaunchDarkly. In fact, I'd go so far as to say that our [26 SDKs and counting]() are critical features of the tool, offering a whole additional layer of functionality on top of the product that ends up being indispensible. In this post, I want to explore some of those features to give you a better sense of the value packed into each SDK.

https://docs.launchdarkly.com/sdk/features

https://launchdarkly.atlassian.net/wiki/spaces/ENG/pages/15466516/SDK+consistency+tracker

As regards the difference between SDKs and going direct to the API, I'd say that the SDKs implement:
a carefully designed caching and streaming system that ensures most flags are evaluated instantaneously in a non-blocking manner
event recording and buffering to track how flags are used
(on the server-side) a full flag evaluation system so that targeting and other rules can be evaluated for all possible users
integration with local persistent stores for caching and offline usage (on mobile this happens automatically)
simplified configuration for LD Relay and other proxies