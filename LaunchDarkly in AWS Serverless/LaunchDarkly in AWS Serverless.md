# LaunchDarkly in AWS Serverless

## LaunchDarkly in Lambda

The good news is that integrating LaunchDarkly into Lambda functions is pretty simple. You'll need to install and configure the [relevant SDK for your language](https://docs.launchdarkly.com/sdk). Here's one way to do this using the [LaunchDarkly Server-Side SDK for Node.js](https://docs.launchdarkly.com/sdk/server-side/node-js): 

1. Create a new Lambda from scratch using the Node.js 16x runtime in AWS.
2. Pull the function code down locally, for example by clicking the "Actions" drop down and choosing "Export").
3. Within the project files locally, install the Node SDK via `npm install launchdarkly-node-server-sdk`.
4. Require and configure the SDK. This can occur outside of the handler so that the SDK is not reinitialized every time the function is run:
```javascript
const LaunchDarkly = require('launchdarkly-node-server-sdk')
const client = LaunchDarkly.init(process.env.LAUNCHDARKLY_SDK_KEY)
```
5. Wait for the client to initialize and then start getting flag values. This must occur within the handler. Note that the below example just passes a "anonymous" as the user key, but you can, of course, pass more detailed user information that is usable for targeting.
```javascript
await client.waitForInitialization();
const flagValue = await client.variation("api-version", { key: "anonymous" }, false);
```
6. Re-upload the updated function, for example by zipping it up and choosing the "Upload from" and then ".zip file" option in the AWS Lambda console.
7. Set the `LAUNCHDARKLY_SDK_KEY` environment variable by going to the "Configuration" tab and choosing "Environment variables". You can obtain your key within the LaunchDarkly flag dashboard by clicking `Command/Ctrl + K` and then choosing "Copy SDK key for the current environment" and finally "Server-side key".
8. Test your function.

You should now be able to retrieve flag data within your function and alter the logic of the function accordingly.

### Using a Lambda Layer

You can repeat the above process for every Lambda you've deployed that needs to use LaunchDarkly, but there's an easier way to ensure that all your Lambda functions have quick and easy access to LaunchDarkly, and that's by creating a [Lambda Layer](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html). The process of doing so is very similar to the above.

Here's how to do it for the Node.js runtime:

1. Create a local folder and, within that folder, create a `nodejs` subdirectory. If you are using a different language runtime,  you can find the relevant [location for dependencies here](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html#configuration-layers-path).
2. Within the `nodejs` directory, run `npm install launchdarkly-node-server-sdk`.
3. Compress the contents of your layer directory (i.e. the `nodejs` directory) but not the containing directory. For example, if the path to the project and `nodejs` folder is `/layer/nodejs`, ensure your zip does not contain the outer `/layer` directory.
3. Go to the [Layers page](https://console.aws.amazon.com/lambda/home#/layers) within the AWS Lambda console and click "Create layer".
4. Give the layer a name. For example, `launchdarkly-node-sdk` and choose Node 16.x as the supported runtime. Finally, upload the zip file containing your layer.

![creating a Lambda layer](create-layer.png)

Now that you have the layer set up, it's relatively easy to use in any Lambda function that requires the LaunchDarkly SDK.

1. In the AWS Lambda console, open the function that you want to add the layer to and scoll down to the Layers section at the bottom of the page. Choose "Add a layer".
2. Select the "Custom layers" option and then, from the dropdown, choose your "launchdarkly-node-sdk" layer. Click "Add".

That's it! Now your Lambda can use the SDK without needing to manually install it or upload the `npm_modules` directory (if this is the only npm dependency you have, even the `package.json` and `package-lock.json` are no longer needed). However, the code within your Lambda to initialize the SDK and get flag values will remain the same.

## Caching Data in a Data Store

Behind the scenes LaunchDarkly does a lot of things to make retrieving flag values incredibly fast, including [delivering flags from the edge](https://launchdarkly.com/blog/flag-delivery-at-edge/), caching flag data locally and streaming updates. This means that getting the SDK client initialized and retrieving flag data takes under 25 milliseconds.

However, using a data store can help speed up the process by allowing you to get the latest flag data without ever needing to make an external call outside AWS. For example, LaunchDarkly's SDKs make it simple to configure [DynamoDB](https://docs.launchdarkly.com/sdk/features/storing-data/dynamodb?q=dynamo) as a data store.

First, you'll need to add the DynamoDB extension for the language you are building your Lambda in. For Node.js, that would mean installing the add-on via npm (`npm install launchdarkly-node-server-sdk-dynamodb`) or adding it to your Lambda Layer and then requiring it.

```javascript
const LaunchDarkly = require("launchdarkly-node-server-sdk");
// The SDK add-on for DynamoDB support
const {
  DynamoDBFeatureStore,
} = require("launchdarkly-node-server-sdk-dynamodb");
```

Next, you'll need to create an instance of a store object that can be passed as a configuration option to LaunchDarkly. You'll need the DynamoDB table name. You can also specify the [caching behavior](https://github.com/launchdarkly/node-server-sdk-dynamodb#caching-behavior) if needed.

Within our configuration options, we'll want to specify `useLdd` as true. This will launch the client in [daemon mode](https://docs.launchdarkly.com/sdk/features/relay-proxy-configuration/daemon-mode). What this means is that the client will use DynamoDB as the source of truth for flag data rather than calling LaunchDarkly.

```javascript
const store = DynamoDBFeatureStore(process.env.DYNAMODB_TABLE);
// useLdd launches the client in daemon mode where flag values come
// from the data store (i.e. dynamodb)
const options = {
	featureStore: store,
	useLdd: true,
};
const client = LaunchDarkly.init(process.env.LAUNCHDARKLY_SDK_KEY, options);
```

From this point on, you can get flag values as normal, but the LaunchDarkly client will only go to DynamoDB. You may be wondering, how do I get the latest flag data into DynamoDB? By default, the SDK client will cache all of the values as soon as it is initialized with a data store, but if you're in daemon mode, that won't happen.

One solution is to create a separate function that will simply initialize the data store and then the SDK client (not in daemon mode), allowing it to automatically synchronize flag data. This function can then be called whenever a flag is changed via LaunchDarkly's webhook integration. You can see that solution discussed in detail on our guide on [using LaunchDarkly in serverless environments](https://docs.launchdarkly.com/guides/infrastructure/serverless).

This works, but there's a potentially better solution depending on your needs in our Relay Proxy. So let's explore that next.

## Using the Relay Proxy in AWS

https://github.com/solve-hq/LaunchDarkly-relay-fargate

## Syncing LaunchDarkly Events

Using flush. Closing the connection.

Additional resources:

https://docs.launchdarkly.com/guides/infrastructure/serverless
