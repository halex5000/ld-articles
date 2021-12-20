# Using the LaunchDarkly EdgeSDK with CloudFlare

Playground: https://cloudflareworkers.com/

Examples: https://developers.cloudflare.com/workers/examples

https://www.smashingmagazine.com/2019/04/cloudflare-workers-serverless/

## Getting Set Up

Install Wrangler https://developers.cloudflare.com/workers/cli-wrangler/install-update

configure with your Cloudflare account using `wrangler login` or follow the instructions here https://developers.cloudflare.com/workers/cli-wrangler/authentication

We're going to use Workers Sites https://developers.cloudflare.com/workers/platform/sites/start-from-scratch (see note) to publish a simple static site built with Hugo

> While Wrangler 2 beta supports a new, streamlined Workers integration with Pages, it does not currently allow for providing a custom Webpack configuration, wich is required for LaunchDarkly's Cloudflare Edge SDK.


### Cloudflare's HTMLRewriter Class

The examples below make use of a a powerful feature that Cloudflare Workers provides called [HTMLRewriter](https://developers.cloudflare.com/workers/runtime-apis/html-rewriter). HTMLRewriter is a JavaScript class that you can leverage within Cloudflare worker code to modify the content of the response being sent back to the user. This allows you to do things like modify the page's HTML or change text in the response. To better understand the code in the examples that follow, let's cover some of the basics of the HTMLRewriter. 

A new instance of the HTMLRewriter class can be constructed as follows:

```javascript
const rewriter = new HTMLRewriter();
```

An instance of HTMLRewriter provides two functions:

1. `on()` listens for any selected elements on the page. Elements are selected via [selectors](https://developers.cloudflare.com/workers/runtime-apis/html-rewriter#selectors) that offer a subset of standard [CSS selectors](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors) commonly used for selecting elements with the document object model (DOM). Each matching element is passed to the element handler that you define.
2. `onDocument()` responds to the entire HTML document, passing the contents of that document to a document handler that you specify.

Corresponding to the above, there are two types of handlers:

1. [Element Handlers](https://developers.cloudflare.com/workers/runtime-apis/html-rewriter#element-handlers) specify the code that will run on each matching element returned by the selector specified in `on()`. This can be used to add, update or remove matching elements and content from within the HTML response.
2. [Document Handlers](https://developers.cloudflare.com/workers/runtime-apis/html-rewriter#document-handlers) specify the code that runs when the entire HTML document is received. This can be used to modify the doctype, modify the text or add code that runs at the end of the document.

Two of the below examples will make use of element handlers to modify the HTML response with a Cloudflare Worker before it is ever received by the end user. If you'd like to know more about HTMLRewriter, check the [Cloudflare docs](https://developers.cloudflare.com/workers/runtime-apis/html-rewriter).

## Bootstrapping client-side flag values

A persistent problem with modifying the client UI on the web using JavaScript is the delay between when a UI element is initially rendered and when the update runs in the script. This causes when can be called a "flash of initial content", where the initial rendering flashes on screen before it gets updated. A common example of this is login/sign up links briefly rendering before getting updated with the logged in user's information.

Imagine a scenario using a LaunchDarkly flag to enable or disable a feature within the browser UI. You definitely do not want the feature to display, however briefly, before disappearing. This could cause confusion and possibly frustration on the part of the user. While LaunchDarkly's client SDKs provide tools caching in LocalStorage to minimize these types of issues, the nature of how JavaScript runs in the browser means that any fully client-side solution cannot completely eliminate the delay. Cloudflare Workers will allow us to eliminate that delay by directly injecting our client-side flag values into the HTML before the request is ever received by the browser.



this will eliminate the flash or small rendering delay that you can see when using normal client side rendering

https://github.com/launchdarkly/launchdarkly-cloudflare-worker-template/blob/main/index.js

https://docs.launchdarkly.com/guides/platform-specific/static-sites#bootstrapping-the-client


## Modifying content at the edge

https://developers.cloudflare.com/workers/examples/ab-testing

https://developers.cloudflare.com/workers/examples/fetch-html

Attach the worker to a route: https://developers.cloudflare.com/workers/platform/routes

## Modifying the Response Headers for a Request

## Conditional Response

https://developers.cloudflare.com/workers/examples/conditional-response


## Conditional Routing



## Rewrite Links

https://developers.cloudflare.com/workers/examples/rewrite-links



## Header values based on flags