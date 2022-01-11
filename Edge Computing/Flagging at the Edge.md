# Flagging at the Edge - Combining LaunchDarkly with Edge Functions

Serverless computing has changed the way we approach application development. No longer did we have to maintain server farms to host and manage our applications or to scale applications by physically adding additional servers. Building serverless applications means that I don't need to worry about the servers or the scaling, that's handled by the cloud provider's platform.

However, applications that rely on serverless APIs often make a lot of API calls. While the client running the application may be on one side of the world, my API may be sitting on servers sitting in an entirely different region across the globe. This can lead to bottlenecks and latency in delivering data to the client. Edge computing aims to solve those problems by deploying critical portions of application code onto servers across multiple regions around the globe, allowing you to reduce or eliminate the latency for many aspects of your application.

In this post, you'll explore a couple of popular edge function solutions and see how integrating them with LaunchDarkly flags can allow for some truly powerful solutions.

## How the serverless edge functions work

One of the key aspects of most edge computing offerings from cloud providers is the edge function. A typical serverless function is deployed to a single region on the cloud provider's platform. However, edge functions are typically automatically replicated across multiple regions globally. This means that any call to the function will hit the region closest to the client, making these functions even faster than a typical serverless function.

Two popular solutions for edge functions are:

* [AWS Lambda@Edge](https://aws.amazon.com/lambda/edge/) – This is AWS's offering for globally replicated Lambda functions. AWS also offers [Cloudfront functions](https://aws.amazon.com/blogs/aws/introducing-cloudfront-functions-run-your-code-at-the-edge-with-low-latency-at-any-scale/) that are deployed at the CDN level, but they are intentionally limited in what they can do.
* [Cloudflare Workers](https://workers.cloudflare.com/) – Every serverless function on Cloudflare is an edge function. They are deployed and replicated at the CDN level across Cloudflare's global CDN network. However, unlike Cloudflare Functions, they are not limited in their capabilities due to being deployed on the CDN.

You can use LaunchDarkly to do things like a percentage rollout of changes or A/B testing without any client-side code becuase the flags are evaluated and the content is modified before the client ever receives the response. Let's explore how you can use LaunchDarkly within both Lambda@Edge and Cloudflare Workers.

## Using LaunchDarkly with Lambda@Edge

For the most part, a Lambda@Edge function is the same as a regular Lambda function, although there are some limitations on region, environment variables and runtime. You can read more about the [restrictions here](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/edge-functions-restrictions.html#lambda-at-edge-function-restrictions).

Thus, you can use LaunchDarkly within a Lambda@Edge function just as you would inside a regular Lambda function, using the [LaunchDarkly server-side Node SDK](https://docs.launchdarkly.com/sdk/server-side/node-js), which can be installed via npm:

```bash
npm install launchdarkly-node-server-sdk
```

The first thing you'll need to do is require the library and initialize the client. Note that due to the restriction on using environment variables within a Lambda@Edge function,

```javascript
const LaunchDarkly = require("launchdarkly-node-server-sdk");
const client = LaunchDarkly.init("<LAUNCHDARKLY_SDK_KEY>");
```

```javascript
await client.waitForInitialization();
let viewBetaSite = await client.variation(
  "rebrand",
  { key: event.Records[0].cf.request.clientIp },
  false
);
console.log(`LaunchDarkly returned ${viewBetaSite}`);
```

## Using LaunchDarkly with Cloudflare Workers