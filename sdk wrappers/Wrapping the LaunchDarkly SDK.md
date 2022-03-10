# Wrapping the LaunchDarkly SDK

LaunchDarkly provides a [huge number of SDKs](https://launchdarkly.com/features/sdk/) for almost any language or platform you can think of using. These SDKs don't just offer an easy-to-use means of connecting your application to LaunchDarkly's API, but they also offer some really important additional functionality. For example, some server-side SDKs offer streaming of flag chages in language that support it, while some client-side SDKs allow for local caching of flag values in order to speed up the initial flag evaluation.

While the SDKs are pretty straightforward to use, you're still likely to encounter a number of situations where you may prefer to create your own custom wrapper around the API provided by the SDK. In this article, I want to explore some scenarios and examples for doing that. The goal isn't to give you copy and paste wrapper options for every SDK – that'd be an awfully long article – but rather to offer some guidance on why you might create one and how you might approach it.

### The Example

To help illustrate the discussion in this article, I've created a simple Node.js Express project with wrappers for both the frontend browser client and the backend Node server environment. The example is intended only as guidance and not meant to be prescriptive. The specific implementation of your own SDK wrappers, should you choose to create them, depends on the needs of your application.

## Why do I need a wrapper?

Well, firstly, you might not. Using the SDK directly throughout your application is a perfectly valid option in many cases. However, here are some reasons why you might be considering creating a custom wrapper:

* **To encapsulate all SDK-specific interaction to a single library** – Rather than having SDK code spread throughout your codebase, it can be useful to keep it all in one place. This is useful for maintenance purposes but also has the added benefit that it helps prevent repeatedly instantiating the SDK client when it's not necessary, which can improve overall performance.
* **To simplify SDK usage** – While the SDK API is designed to be straightforward and easy to use, it is also designed to meet the varying needs of a wide array of customers and applications. You may find that there are some quick shortcuts you can apply that are tailored to the way you use the SDK.
* **To prevent accidental bugs** – If you're an organization with many developers accessing flags, it's important to have a consistent naming strategy for flags. However, that does not prevent typos from causing potential problems. Your wrapper can be designed to help you maintain naming standards and help prevent misspelled flag keys in the code.
* **To add domain-specific business logic around flag usage** – Perhaps your organization has additional business rules or logic around flags that you need to enforce application-wide. Having a wrapper can help you encapsulate that logic to help ensure it is properly applied wherever flags are used.

Let's look at a few of these in more depth.

## Encapsulate interaction

I probably don't need to sell you on the general benefits of encapsulation in your application code. You're probably well aware of how it makes debugging and maintanence easier because the code only ever needs to be updated in a single location. Or how it can help prevent errors by making the code easier to understand and hiding some of the lower level complexity.

Of course, all of these benefits apply to your LaunchDarkly SDK wrapper. By keeping all of your interaction with LaunchDarkly within a single location in your application, you'll ensure that everyone connects the same way, have a single place to update if any changes are needed and potentially reduce the learning curve towards using flags, as each developer doesn't have to learn the details of the underlying SDK.

However, there's an added benefit. The SDK client needs to be initialized before it can be used. While the cost of initialization is small (under 25ms), there's certainly no reason to needlessly incur it more than necessary. Having flag calls run through a wrapper can help ensure that this doesn't happen. Let's look at a simple example.

The code below is a simple wrapper class just for the SDK client. It has two methods:

* `#initialize()` is a private method handles initializing the client and waiting for that initialization to complete.
* The `getClient()` method checks to see if the client has been initialized and, if not, initializes it.

```javascript
const LaunchDarkly = require("launchdarkly-node-server-sdk");
require("dotenv").config();

module.exports = class Client {
  constructor() {
    this.client = null;
  }

  async #initialize() {
    const client = LaunchDarkly.init(process.env.LAUNCHDARKLY_SDK_KEY);
    await client.waitForInitialization();
    return client;
  }

  async getClient() {
    if (this.client) return this.client;

    return await this.#initialize();
  }
};
```

This class is designed to be used as a singleton, helping prevent developers using it from reinstantiating the SDK client when not necessary. In fact, the developer does not need to worry about whether the client has or has not been initialized – calling `getClient()` will handle all that for them.

## Simplify



```javascript
const LaunchDarkly = require("launchdarkly-js-client-sdk");
import { LAUNCHDARKLY_CLIENT_ID } from "env";

let launchDarklyClient;

async function initialize(user) {
  if (!user) {
    user = {
      key: "anonymous",
    };
  }
  const client = LaunchDarkly.initialize(LAUNCHDARKLY_CLIENT_ID, user);
  await client.waitForInitialization();
  return client;
}

async function getClient() {
  if (launchDarklyClient) return launchDarklyClient;
  return (launchDarklyClient = await initialize());
}

export async function getFlagValue(key, fnChangeListener) {
  const client = await getClient();
  let flagValue;

  flagValue = await client.variation(key, false);

  if (fnChangeListener) {
    client.on("change:" + key, fnChangeListener);
  }
  return flagValue;
}
```

## Prevent Errors

```javascript
module.exports = class User {
  constructor(
    client,
    key,
    secondary,
    ip,
    email,
    firstName,
    lastName,
    anonymous,
    os,
    device
  ) {
    this.client = client;

    if (!key) key = "anonymous";

    this.user = {
      key: key,
      secondary: secondary,
      ip: ip,
      email: email,
      firstName: firstName,
      lastName: lastName,
      anonymous: anonymous,
      os: os,
      device: device,
    };
  }

  async getFlagValue(key, callback) {
    let flagValue;

    if (!this.client) throw new Error("Client not defined");

    flagValue = await this.client.variation(key, this.user);

    if (callback) {
      this.client.on("update:" + key, async (keyName) => {
        const flagValue = await this.client.variation(keyName.key, this.user);
        callback(flagValue);
      });
    }
    return flagValue;
  }

  async closeClient() {
    if (this.client) {
      await this.client.close();
      this.client = null;
    }
  }
};
```