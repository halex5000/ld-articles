# Getting Started with Svelte and LaunchDarkly

If you're a frontend and/or JavaScript developer, chances are you've heard of [Svelte](https://svelte.dev/). In fact, if the latest [State of JavaScript survey](https://2021.stateofjs.com/en-US/libraries/front-end-frameworks) is to be believed, there's a good chance that you want to give it a try, as it has remained the framework JavaScript developers are most interested in two years in a row.

In case you haven't heard of Svelte, it's a frontend framework like React and Vue, but it has a very different approach. Whereas React and Vue do most of their work in the browser via included scripts, Svelte pushes most of its work to the build, performing most of its work during the compile. By doing so, it aims to reduce the size of the JavaScript bundle, which can make your application lighter and also potentially improve its performance.

The good news is that, since Svelte is just JavaScript, you don't need any framework-specific libraries to use it with LaunchDarkly – you can leverage the existing JavaScript libraries. Let's see how.

## Working with Client-side Svelte Code

There is nothing preventing you from adding feature flags to your heart's content in client-side code in Svelte using [LaunchDarkly's JavaScript SDK](https://docs.launchdarkly.com/sdk/client-side/javascript). It'll just work as you expected, out of the box.

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
    console.log(showButton);
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

For my sample project, I've created a simple wrapper for my interactions with the JavaScript SDK. This allows me to keep the code for initializing the SDK using the client-side ID in a single location and even create some shortcuts. For instance, to get a single flag's value, a component only needs to call `getFlagValue`, the library will take care of initializing the SDK if necessary and even offers a shortcut for adding a change listener to the passed flag key.



