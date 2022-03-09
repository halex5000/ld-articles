# Wrapping the LaunchDarkly SDK

LaunchDarkly provides a [huge number of SDKs](https://launchdarkly.com/features/sdk/) for almost any language or platform you can think of using. These SDKs don't just offer an easy-to-use means of connecting your application to LaunchDarkly's API, but they also offer some really important additional functionality. For example, some server-side SDKs offer streaming of flag chages in language that support it, while some client-side SDKs allow for local caching of flag values in order to speed up the initial flag evaluation.

While the SDKs are pretty straightforward to use, you're still likely to encounter a number of situations where you may prefer to create your own custom wrapper around the API provided by the SDK. In this article, I want to explore some scenarios and examples for doing that. The goal isn't to give you copy and paste wrapper options for every SDK – that'd be an awfully long article – but rather to offer some guidance on why you might create one and how you might approach it.

# Why do I need a wrapper?

Well, firstly, you might not. Using the SDK directly throughout your application is a perfectly valid option in many cases. However, here are some reasons why you might be considering creating a custom wrapper:

* To isolate all SDK-specific interaction to a single library. all interaction in one place in case you want to change later. prevents instantiating client when not necessary.
* prevent bugs caused by typos. keep standards around naming or even prevent misspelled flags
* 