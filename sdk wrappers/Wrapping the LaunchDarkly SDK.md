# Wrapping LaunchDarkly

LaunchDarkly provides [a lot of SDKs](https://launchdarkly.com/features/sdk/) for a broad list of languages, frameworks and platforms. These SDKs don't just offer an easy-to-use means of connecting your application to LaunchDarkly's API, but they also offer some really important additional functionality. For example, server-side SDKs offer streaming of flag changes for languages that support it, while some client-side SDKs allow for local caching of flag values in order to speed up the initial flag evaluation.

Even though the SDKs are straightforward to use, you're still likely to encounter a number of situations where you may prefer to create your own custom wrapper around the API provided by the SDK. In this article, I want to explore some scenarios and examples for doing that. The goal isn't to give you "copy-and-paste" wrapper options for every SDK – that'd be an awfully long article – but rather to offer some guidance on why you might create one and how you might approach it.

### The Example

To help illustrate the discussion in this article, I've created a simple Node.js Express project with wrappers for both the frontend browser client and the backend Node server environment. The example is intended only as guidance and not meant to be prescriptive of how you should create your own wrappers. The specific implementation of your SDK wrappers, should you choose to create them, depends on the needs of your application. You can access the example code [on GitHub](https://github.com/remotesynth/launchdarkly-sdk-wrapper-example).

## Why do I need a wrapper?

Well, firstly, you might not. Using the SDK directly throughout your application is a perfectly valid option in many cases and we provide many examples in our documentation on how to accomplish this. However, here are some reasons why you might be considering creating a custom wrapper:

* **To encapsulate all SDK-specific interaction to a single library** – Rather than having SDK code spread throughout your codebase, it can be useful to keep it all in one place. This is useful for maintenance purposes but also has the added benefit that it helps prevent repeatedly instantiating the SDK client when it's not necessary, which can improve overall performance.
* **To simplify SDK usage** – While the SDK API is designed to be straightforward and easy to use, it is also designed to meet the varying needs of a wide array of customers and applications. You may find that there are some quick shortcuts you can apply that are tailored to the way you use the SDK.
* **To prevent accidental bugs** – If you're an organization with many developers accessing flags, it's important to have a consistent naming strategy for flags. However, that does not prevent typos from causing potential problems. Your wrapper can be designed to help you maintain naming standards and help prevent accidental errors.
* **To add domain-specific business logic around flag usage** – Perhaps your organization has additional business rules or logic around flags that you need to enforce application-wide. Having a wrapper can help you encapsulate that logic to help ensure it is properly applied wherever flags are used.

The last one is very specific to your application use case, so let's look at the first three of these in more depth.

## Isolate SDK Code

By keeping all of your interaction with LaunchDarkly within a single location in your application, you'll streamline the SDK configuration process, ensure that everyone connects the same way, have a single place to update if any changes are needed and potentially reduce the learning curve towards using flags, as each developer doesn't have to learn the details of the underlying SDK. This can also help adoption of LaunchDarkly within your teams by creating a set of code that can be reused across not just one application but many.

There's another important benefit. The SDK client needs to be initialized before it can be used. While the cost of initialization is small (under 25ms), you do not want to needlessly incur it more than necessary. Having flag calls run through a wrapper can help ensure that this doesn't happen. Let's look at an example.

The code below is an example of a very basic wrapper class just for the server-side Node.js SDK client. It has two methods:

* `#initialize()` is a [private method](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_class_fields) that handles initializing the client and waiting for that initialization to complete.
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

Don't get me wrong, I find LaunchDarkly's SDK APIs easy to use and straightforward. Still, that doesn't mean that you can't find some nice shortcuts. How you wrap the SDK to simplify your usage of it will depend largely on how you utilize LaunchDarkly and how you use flags within your application, so my shortcuts may not be your shortcuts.

Creating these simplified shortcuts can also help you standardize how your team interacts with the SDK by turning a series of steps into a single one. Let's look at a couple examples of this that I have used in my code.

On a client-side application, it is common that you'd want to add a listener to see if a flag changes after getting the initial flag state. LaunchDarkly will notify your application of any flag changes (within 200ms of the change being made), but you have to be listening for them. This is usually a two-step process. First, you get the initial flag state. Second, you add a change listener to that flag key and manage any rerendering as needed.

In this client-side JavaScript SDK wrapper, I've converted that to a single step by allowing you to pass a change listener to the `getFlagValue()` function. The function will pass back the flag value but also set a change listener for the passed flag key at the same time.

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

  // if a change listener was passed,
  // set it to listen to changes for this key
  if (fnChangeListener) {
    client.on("change:" + key, fnChangeListener);
  }
  return flagValue;
}
```

The wrapper has other tools to simplify the SDK usage including automatically initializing the SDK client if it isn't yet initialized when you attempt to get a flag value and setting an anonymous user key if no user is passed. This is an example of where your own usage and preferences come in. For instance, you may instead want to throw an error if no user is passed to ensure that the developer always passes a user and cannot accidentally set an anonymous key.

Now when I use my wrapper, I can intialize the client, get the flag value and set a change listener all in a single line of code. In the below code, I:

* Define the variable that will hold the flag button.
* Create a setter function that will be used to set the variable and can function as my callback.
* Get the value of the flag with the key `show-button` and pass the `setShowButton()` function as a callback. Since `getFlagValue()` is an async function that returns a promise, I then call the `setShowButton()` function to set the initial state once the promise has been resolved.

```javascript
let showButton;

function setShowButton(val) {
  showButton = val;
}

// get the flag value and pass a change listner
// when the promise resolves, call the setter
getFlagValue("show-button", setShowButton).then(setShowButton);
```
## Prevent Errors

With any tool, there is always a delicate balance between making something easy to use and making it too easy to make mistakes. I think the SDKs strike the right balance, but you may want to add additional assurances against errors and mistakes by utilizing your wrapper.

For instance, you may want to ensure that the user context is always set and remains consistent so that the user always gets the appropriate flag state. You also may want to ensure that your developers don't pass a flag key with a typo, which, if they've told the SDK to set a default value, could result in incorrect flag results.

The following example is a server-side Node.js `User` class. It is designed to work with the `Client` class shown earlier whereby you'd pass the user an instance of the initialized client. The reason for separating the two, unlike in the client-side wrapper above, is that the client-side JavaScript SDK expects the SDK client to be initialized with a user while the server-side Node.js SDK expects the user passed on each call. By wrapping everything in a `User` class, we can help prevent issues that may occur if the developer passes the wrong user context when requesting a flag value.

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

    // a key is the only required attribute
    // set it to anonymous if nothing was passed
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

    flagValue = await this.client.variation(key, this.user, false);

    // server side change listeners don't pass the key value
    // so if a callback is passed, create an update function that
    // gets the flag value and passes that back to the listener
    if (callback) {
      this.client.on("update:" + key, async (keyName) => {
        const flagValue = await this.client.variation(keyName.key, this.user, false);
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

You may have noticed that the `getFlagValue()` method also provides a simplified way of adding a callback on the change event for a flag key. Since Node.js supports streaming, all flag changes are streamed to the server, allowing us to receive and react to flag changes in real-time. This wrapper can make adding that callback easier for the developer.

Using this wrapper involves creating an instance of the `Client` class and passing the initialized client to the `User`. Then, I can get the flag variation for that user and pass the result to a setter function. That same setter function can be passed as the callback for any changes to that flag key.

```javascript
// client is a singleton
const client = new ldServer();

// each user would be initialized with their info
let user = new ldUser(await client.getClient());
setShowButton(await user.getFlagValue("show-button", setShowButton));
```

A different strategy, that is used within LaunchDarkly's own codebase, is to prevent errors caused by typos or misuse of flag keys, while also providing code hinting that makes it easier to know what flags are available to use.

For example, I could modify the client-side wrapper to make add constants that represent the available flag values.

```javascript
export const showButton = getFlagValue('show-button');
```

This strategy would negate the usefulness of the automatic callback function in the manner it is currently implemented, but would allow me to get code hinting within my code editor so that I could simply call `client.showButton` to get the current flag value.

## Find your own wrap

The goal here is not for you to use my wrappers, though you are free to borrow any aspect that you find useful. They are designed to meet the needs of the applications I'm currently building using LaunchDarkly. Nonetheless, the broad principles remain the same, regardless of what language you are implementing your wrapper in (we currently have [24 SDKs](https://docs.launchdarkly.com/sdk#available-sdks) for different languages, frameworks and platforms). Create the wrapper however you best works for you and your team, but use it to encapsulate interaction, simplify your codebase, prevent errors and add domain-specific business logic. That's a wrap!