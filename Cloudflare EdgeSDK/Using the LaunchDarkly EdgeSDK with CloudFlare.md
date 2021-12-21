# Using the LaunchDarkly EdgeSDK with CloudFlare

Playground: https://cloudflareworkers.com/

Examples: https://developers.cloudflare.com/workers/examples

https://www.smashingmagazine.com/2019/04/cloudflare-workers-serverless/

## Getting set up

For this guide, we've created a sample CloudFlare Workers project that demonstrates several uses of flags within a Worker. You can find the finished code [on GitHub](https://github.com/remotesynth/cfworkers-ld). You can use this code as reference as you work through steps in the guide. You can view the finished page running on Cloudflare Workers [here](https://cfworkers-ld.remotesynth.workers.dev/).

If you would like to follow along creating the project in this guide, begin by cloning the site assets [in this repository](https://github.com/remotesynth/cfworkers-ld-assets). In order to compile the site, you'll need to [install Hugo](https://gohugo.io/getting-started/installing/), a static site generator built in Go. The easiest way to install Hugo is using Homebrew on Mac:

```bash
brew install hugo
```

Or via Chocolatey on Windows:

```bash
choco install hugo -confirm
```

### Setting up Cloudflare CLI

The quickest way to get set up is to use [Cloudflare's Worker CLI](https://developers.cloudflare.com/workers/cli-wrangler/install-update) called Wrangler via npm:

```bash
npm install -g @cloudflare/wrangler
```

Or via Yarn:

```bash
yarn global add @cloudflare/wrangler
```

Once it is installed, you'll need to use the `wrangler login` command, which will open a browser window allowing you to log into your Cloudflare account to authenticate the CLI. For additional CLI authentication methods, you can reference the [Wrangler docs](https://developers.cloudflare.com/workers/cli-wrangler/authentication).

### Setting up local development for Cloudflare Workers

This guide uses [Workers Sites](https://developers.cloudflare.com/workers/platform/sites) that allow you to deploy an application buit with a static site generator (SSG) or front end framework to a worker. You can follow the [instructions for starting from an existing project](https://developers.cloudflare.com/workers/platform/sites/start-from-existing) to publish a simple static site built with the [Hugo SSG](https://gohugo.io).

> The Wrangler 2 beta supports a new, streamlined Workers integration with Pages, Cloudflare's solution for deploying sites. However, it does not currently allow for providing a custom Webpack configuration, which is required for LaunchDarkly's Cloudflare Edge SDK.

Open the terminal in the folder containing the site assets from GitHub and enter the following command:

```bash
wrangler init --site my-static-site
```

This should have created a `wrangler.toml` and a `workers-site` directory within the project. Open the `wrangler.toml` and add the following line that tells the worker to find the site files in the `/public` folder, which is where Hugo outputs them by default:

```toml
site = { bucket = "./public" }
```

While we haven't actually created a Worker yet, we can run the site locally using Wrangler. The first step is to build the site using hugo and then run a local server with Wrangler.

```bash
hugo
wrangler dev
```

### Setting up the LaunchDarkly Cloudflare integration

### Cloudflare's HTMLRewriter class

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

Within your Cloudflare Worker file, you'll need to first instantiate an instance of the HTMLRewriter class.

```
const rewriter = new HTMLRewriter();
```

You'll want to inject these values in the HTML `<head>` so that they are available immediately before any of the DOM or JavaScript gets processed by the browser engine. You can do this by having the HTMLRewriter listen for the head element.

```javascript
rewriter.on("head", new FlagsStateInjector());
```

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