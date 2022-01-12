# Flagging at the Edge - Combining LaunchDarkly with Edge Functions

Serverless computing has changed the way we approach application development. No longer do we have to maintain server farms to host and manage our applications, or to scale applications by physically adding additional servers. Building "serverless applications" means that I don't need to worry about the servers or the scaling, that's handled by the cloud provider's platform.

However, applications that rely on serverless APIs often rely on a lot more API calls than traditional systems might. While the client running the application may be on one side of the world, my API may be sitting on servers sitting in an entirely different region across the globe. This can lead to performance bottlenecks and latency in delivering data to the client. Edge computing aims to solve those problems by deploying critical portions of application code onto servers across multiple regions around the globe, allowing you to reduce or eliminate the latency for many aspects of your application.

In this post, we'll explore a couple of popular edge function solutions and see how integrating them with LaunchDarkly flags can allow for some truly powerful solutions.

## How the serverless edge functions work

One of the key aspects of most edge computing offerings from cloud providers is the concept of an "edge function". A typical serverless function is deployed to a single region on the cloud provider's platform. However, edge functions are typically replicated (automatically) across multiple regions globally. This means that any call to the function will hit the region closest to the client, making these functions even faster than a typical serverless function would perform.

Two popular solutions for edge functions are:

* [AWS Lambda@Edge](https://aws.amazon.com/lambda/edge/) – This is AWS's offering for globally replicated Lambda functions. AWS also offers [Cloudfront functions](https://aws.amazon.com/blogs/aws/introducing-cloudfront-functions-run-your-code-at-the-edge-with-low-latency-at-any-scale/) that are deployed at the CDN level, but they are intentionally limited in what they can do.
* [Cloudflare Workers](https://workers.cloudflare.com/) – Every serverless function on Cloudflare is an edge function. They are deployed and replicated at the CDN level across Cloudflare's global CDN network. However, unlike Cloudflare Functions, they are not limited in their capabilities due to being deployed on the CDN.

You can use LaunchDarkly to do things like a percentage rollout of changes or A/B testing without any client-side code, becuase the flags are evaluated and the content is modified before the client ever receives the response. Let's explore how you can use LaunchDarkly within both Lambda@Edge and Cloudflare Workers.

## Using LaunchDarkly with Lambda@Edge

For the most part, a Lambda@Edge function is the same as a regular Lambda function. There are some limitations on region, environment variables and runtime that can come into play in some cases. You can read more about these [restrictions here](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/edge-functions-restrictions.html#lambda-at-edge-function-restrictions).

Thus, you can use LaunchDarkly within a Lambda@Edge function just as you would inside a regular Lambda function using the [LaunchDarkly server-side Node SDK](https://docs.launchdarkly.com/sdk/server-side/node-js), which can be installed via npm:

```bash
npm install launchdarkly-node-server-sdk
```

The first thing you'll need to do is require the library and initialize the client with your credentials. These are available on the [Accounts section of the dashboard](https://app.launchdarkly.com/settings/projects) under the projects tab. You'll need the SDK key for the project and environment you'll be utilizing within the Lambda function. Place this code at the top of your Lambda file, outside the standard event handler.

> Note that due to the restriction on using environment variables within a Lambda@Edge function, you'll need to include your SDK key, so be sure that your code is not checked into a public repository.

```javascript
const LaunchDarkly = require("launchdarkly-node-server-sdk");
const client = LaunchDarkly.init("<LAUNCHDARKLY_SDK_KEY>");
```

Before you can begin using the client to get flag values within the event handler, you'll need to wait for the initialization to complete. The easiest way to do this, in my opinion, is to use `async`/`await`. Thus, the handler needs to be an async function and then you can just `await` the initialization.

```javascript
exports.handler = async (event) => {
  await client.waitForInitialization();  
  ...
}
```

Once the initialization is complete, you are free to get flag values. The example below is getting a variation for the "my-first-flag" flag and using the client IP address a key for the user data. Variations in the LaunchDarkly platform are the different values a flag thats been created might return. You can pass whatever data about the user that you like within the user object. This can be used to target a user through targeting rules you set within the LaunchDarkly dashboard or for doing things like percentage rollouts.

```javascript
let myFirstFlag = await client.variation(
	"my-first-flag",
	{ key: event.Records[0].cf.request.clientIp },
	false
);
console.log(`LaunchDarkly returned ${myFirstFlag}`);
```

You can now use the value of `myFirstFlag` to modify the logic within the Lambda function. For example, perhaps the flag you've implemented is used to determine whether a user gets redirected to a beta version of the site. This logic would happen prior to the client receiving their response. This means that, unlike client-side code, anything that happens within the Lambda@Edge function is completely transparent to the user.

> For more details on working with Lambda and Lambda@Edge, check out our guide on [Using LaunchDarkly with AWS Lambda](https://docs.launchdarkly.com/guides/platform-specific/aws-lambda).

## Using LaunchDarkly with Cloudflare Workers

Cloudflare Workers are serverless functions that work the same as an AWS Lambda function. However, unlike Lambda, which has different offerings for regular serverless functions and edge functions, every Cloudflare Worker is an edge function deployed to Cloudflare's global network of CDNs.

LaunchDarkly offers a direct integration with Cloudflare Workers that will automatically synchronize flag values from your LaunchDarkly project environment to the key value store (KV) associated with your Worker. This means that the Cloudflare Worker always has access to all of the up-to-date flag values with no latency.

> Before you can work with the LaunchDarkly Cloudflare integration, you'll need to walk through the steps laid out in the [Cloudflare integration documentation](https://docs.launchdarkly.com/integrations/cloudflare) to set up the KV and configure the integration.

LaunchDarkly's Cloudflare integration comes with its own SDK that you can install via npm.

```bash
npm install launchdarkly-cloudflare-edge-sdk
```

You'll also need to configure [Wrangler](https://developers.cloudflare.com/workers/cli-wrangler), Cloudflare's Worker CLI, to work with the Edge SDK by following the [instructions here](https://docs.launchdarkly.com/sdk/server-side/node-js/cloudflare-edge-sdk#getting-started).

Within your Worker file, you'll start by importing the SDK and initializing the variable that will hold the client.

```javascript
const { init } = require("launchdarkly-cloudflare-edge-sdk");
let ldClient;
```

A Worker will typically have an event listener that listens for the `fetch` event that is triggered by any incoming HTTP request. You can initialize the LaunchDarkly client using your LaunchDarkly Client ID within the event handler. The client ID is available on the [Accounts section of the dashboard](https://app.launchdarkly.com/settings/projects) under the projects tab. `MY_KV` references the key value store that is defined within the `kv_namespaces` of `wrangler.toml`. For details on how to define that, follow the [instructions here](https://docs.launchdarkly.com/sdk/server-side/node-js/cloudflare-edge-sdk#getting-started). You'll need to wait for the LaunchDarkly client initialization before attempting to get flag values within the Worker.

```javascript
addEventListener("fetch", (event) => {
  event.respondWith(handleEvent(event));
});

async function handleEvent(event) {
  if (!ldClient) {
  	ldClient = init(MY_KV, "<LAUNCHDARKLY_CLIENT_ID>");
  	await ldClient.waitForInitialization();
  }
}
```

Now that everything is initialized, you are ready to start getting flag values. Remember, these are synchronized with the KV which makes getting flag values extremely fast! As an example example, you can use Cloudflare's [HTMLRewriter class](https://developers.cloudflare.com/workers/runtime-apis/html-rewriter) to modify the page header for a page based upon the value of a flag.

```javascript
class H1ElementHandler {
  async element(element) {
    // replace the header text with the value of a string flag
    const headerText = await await ldClient.variation("header-text", { "anonymous" }, false);
    element.setInnerContent(headerText);
  }
}
const rewriter = new HTMLRewriter();
rewriter.on("h1", new H1ElementHandler());
```

Since the HTML response is being modified at the edge, the user will not see any flash of content as they might when this type of change is performed client-side. Right now, the code passes an anonymous user key, but we could also supply identifying information so that only QA testers see the modified header or we perform an A/B test across our users with all of these changes happening transparently to the user. In this example, we might pass the "QA" header to the client and subsequently the user object wtihin LaunchDarkly. We can then create flags that target only those users based on that QA value - creating an easy way to present QA changes in our live environment.

## Take a trip to the edge

Edge computing and edge functions are quickly becoming a standard offering of almost every cloud provider because they offer the capability to move some critical aspects of application logic closer to the end user. This offers them a better and faster experience while also opening up new and unique possibilties in what our applications can do. By integrating feature management and feature flags into our edge functions, we can now quickly and transparently release features, change logic, adjust the HTML response and more with just the flip of a switch in the LaunchDarkly dashboard.
