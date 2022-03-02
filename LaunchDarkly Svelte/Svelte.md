# Getting Started with Svelte and LaunchDarkly

If you're a frontend and/or JavaScript developer, chances are you've heard of [Svelte](https://svelte.dev/). In fact, if the latest [State of JavaScript survey](https://2021.stateofjs.com/en-US/libraries/front-end-frameworks) is accurate, there's a good chance that you want to give it a try. It has remained the framework JavaScript developers are most interested in two years in a row.

In case you haven't heard of Svelte, it's a frontend framework like React and Vue, but it has a very different approach. Whereas React and Vue do most of their work in the browser via included scripts, Svelte pushes this to the build step, performing most of its work during the compile. By doing so, it aims to reduce the size of the JavaScript bundle, which can make your application lighter and also potentially improve its performance.

The good news is that, since Svelte is just JavaScript, you don't need any framework-specific libraries to use it with LaunchDarkly – you can leverage the existing JavaScript libraries. Let's see how.

> This post covers the basics of integrating Svelte and LaunchDarkly. If you are looking for something much more in depth, check out our [guide to using LaunchDarkly with Svelte](https://docs.launchdarkly.com/guides/platform-specific/svelte) in the documentation.

## Working with Client-side Svelte Code

There is nothing preventing you from adding feature flags to your heart's content in client-side code in Svelte using [LaunchDarkly's JavaScript SDK](https://docs.launchdarkly.com/sdk/client-side/javascript). It'll just work as you expected, out of the box using the same code as shown in the SDK documentation.

For example, the below is a basic Svelte component that does the following:

1. Imports the JavaScript SDK.
2. Initializes the client using the Client-side ID and an anonymous user key.
3. When the client is initialized and ready, it gets the value of the `show-button` boolean flag.
4. Sets a listener on the change event for the flag so that any change to the flag value in LaunchDarkly will be immediately reflected on the client.
5. If the flag is true, the button will display within the component. If it is false, the button is hidden.

```html
<script>
  import * as LaunchDarkly from "launchdarkly-js-client-sdk";

  let showButton = false;
  const client = LaunchDarkly.initialize("<LAUNCHDARKLY_CLIENT_ID>", {
    key: "anonymous"
  });
  client.on("ready", () => {
    setShowButton(client.variation("show-button", false));
    client.on("change:show-button", setShowButton);
  });

  function setShowButton(val) {
    showButton = val;
  }
</script>

<style>
  main {
    font-family: sans-serif;
    text-align: center;
  }
</style>

<main>
	<p>This will always show</p>
  {#if showButton}
    <button>Flag is true</button>
  {/if}
</main>
```

If you are building components in SvelteKit, you'll need to add a check to ensure that the code is running in the browser and not on the server by wrapping it in a `browser` check using the built-in [`$app/env` module](https://kit.svelte.dev/docs/modules#$app-env). Otherwise you'll receive compiler errors indicating that browser modules like `window` can't be found.

## Working with Server-side Code in SvelteKit

Just as with client-side code in Svelte, no special library is necessary to work with LaunchDarkly within SvelteKit. You can use the [server-side Node SDK](https://docs.launchdarkly.com/sdk/server-side/node-js).

However, you'll want to ensure that all interactions with the Node SDK are performed server-side only. The easiest way to do this is to isolate your interaction with the SDK in [SvelteKit endpoints](https://kit.svelte.dev/docs/routing#endpoints), which run exclusively server-side. The below example is a basic endpoint that gets the value of the `featured-username` flag and uses that to alter the parameters in an API call.

```javascript
import LaunchDarkly from "launchdarkly-node-server-sdk";

export async function get() {
  const client = LaunchDarkly.init(import.meta.env.VITE_LAUNCHDARKLY_SDK_KEY);
    await client.waitForInitialization();
  const featuredUsername = await client.variation("featured-username", { key: "anonymous"}, false);
  const response = await fetch(
    `https://dev.to/api/articles?username=${featuredUsername}&page=1&per_page=10`
  );
  let posts = await response.json();

  return {
    body: {
      posts,
      featuredUsername,
    },
  };
}
```

In most cases, you won't want to reinitialize the library within every single endpoint that requires a flag. The most straightforward solution to this is to create a shared library that handles the initialization of the SDK and returns the client. Every endpoint that requires a flag value can then include this library to get an instance of the SDK client that has been initialized. You can see an example of this in the [our Svelte  guide](https://docs.launchdarkly.com/guides/platform-specific/svelte).

## Where to Go From Here

The good news is that you can use the existing LaunchDarkly JavaScript and Node SDKs as is within your Svelte project, without any complicated workarounds or framework specific code. The above examples just touched on the most basic implementation. If you are looking for a more in depth tutorial and set of examples, check out our complete [guide to using LaunchDarkly with Svelte](https://docs.launchdarkly.com/guides/platform-specific/svelte) in the documentation.