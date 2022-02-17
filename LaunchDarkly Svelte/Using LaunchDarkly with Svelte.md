# Using LaunchDarkly with Svelte

## Overview

This guide explains how to connect LaunchDarkly with Svelte and SvelteKit on both the client and server side.

Svelte is a  JavaScript toolset that accomplishes many of the same goals as React, updating the user interface via document object model (DOM) updates and managing state. However, unlike React, which performs it's updates via client-side script that is run in the browser, Svelte does much of its processing at build-time. This allows it minimize the JavaScript bundle size and limit the amount of JavaScript required to perform document object model (DOM) updates. The result can make your application lighter and, potentially, faster than a comparable React or Vue-based application.

SvelteKit is an officially supported full-stack framework for building web applications based upon Svelte,  similar to what Next.js provides in relation to React. SvelteKit enables you to write server-side code using Node.js and add file-based routing to a Svelte application.

In this tutorial, we'll see how to use the LaunchDarkly SDKs on both the client and server side within an application built with SvelteKit.

## Setting up

For this guide, we created a simple SvelteKit project that demonstrates how to incorporate LaunchDarkly feature flags within both the backend server-side code as well as the frontend client-side code. The finished project is available [on GitHub](https://github.com/launchdarkly-labs/ld-svelte-basics).

If you'd like to run the project locally, clone the GitHub project and then run:

```bash
npm install
```

This will install all of the project's dependencies, including both Svelte and SvelteKit.

### Which SDK do I use?

Since the example has flags that are evaluated on both the server-side and the client-side, it requires a separate SDK for each.

Server-side code within SvelteKit is run in a Node.js environment. This will require the use of LaunchDarkly's [Node SDK (server-side)](https://docs.launchdarkly.com/sdk/server-side/node-js). This is included in the project's dependencies, but you can install it via npm:

```bash
npm install launchdarkly-node-server-sdk
```

However, client-side code will need to use the [client-side JavaScript SDK](https://docs.launchdarkly.com/sdk/client-side/javascript). There is no need for a framework-specific client-side SDK for Svelte. This SDK is also included in the project's dependencies, but you can install it via npm:

```bash
npm install launchdarkly-js-client-sdk
```

### Server-side versus client-side code in SvelteKit

One of the unique aspects of SvelteKit is that JavaScript within the component can often run on both the client and on the server. This allows it to rehydrate the client without reloading the page from the server. This can cause some confusion when using the client-side JavaScript and server-side Node.js SDKs.

If the server-side Node.js SDK is loaded within an environment that also runs in the browser, this will cause a number of build errors due to the lack of the default Node.js modules like `util`, `fs` and others. The simplest solution is to isolate usage of the Node.js server-side SDK to server-side only code within SvelteKit such as [endpoints](https://kit.svelte.dev/docs/routing#endpoints).

If the client-side JavaScript SDK is loaded within an environment that also runs on the server, it will cause errors because the Node.js environment lacks browser defaults like `window`.  The way around this is to isolate browser-specific code. SvelteKit provides a default `$app/env` module that includes a `browser` value that indicates whether the app is running in the browser or the server. You can isolate browser code using this as shown below:

```javascript
if (browser) {
	// browser specific code
}
```

## Server-side rendering (SSR)

A typical SvelteKit route or page has a `.svelte` file extension and it is paired with a JavaScript file (`.js`) that provides the server-side data for the page. For example, your site's home page would typically be in `/src/routes/index.svelte`. This would contain route specific UI code, JavaScript code and CSS. This may be paired with a file `/src/api/index.js` that would provide the backend to this page, supplying it with whatever data it needs from the server.

> Note that SvelteKit very recently introduced something called "shadow routes". In this case, `/src/routes/index.svelte` would be paired with `/src/routes/index.js`. There is also no need to use the `load` function in `index.svelte` to call the API as it will be called automatically. However, this functionality is very new and still has some bugs, so our example application uses the traditional set up.

### Creating a server-side wrapper

You'll likely want to centralize all calls to the SDK into a single location. This allows you to initialize the SDK in a single location, reuse the instance if it is already initialized and standardize the way your code interacts with the API. The example application includes a basic server-side wrapper for the server-side Node.js SDK.

The server-side wrapper is in the file `/src/lib/launchdarkly/server.js`. The wrapper does not implement all the SDK functionality but has the following functions:

* `initialize()` handles initializing the SDK using the LaunchDarkly SDK Key that is stored in an environment variable via a `.env` file. It then waits for the SDK to initialize and returns the SDK client.
* `getClient()` simply checks to see if the client has already been initialized and, if not, initializes it. In either case, it returns an instance of the SDK client.
* `getFlagValue` will return the value of a LaunchDarkly flag based upon the flag's key and the user data that is passed. If no user data is passed, the function will pass a generic anonymous user key.

```javascript
import LaunchDarkly from "launchdarkly-node-server-sdk";

let launchDarklyClient;

async function initialize() {
  const client = LaunchDarkly.init(import.meta.env.VITE_LAUNCHDARKLY_SDK_KEY);
  await client.waitForInitialization();
  return client;
}

async function getClient() {
  if (launchDarklyClient) return launchDarklyClient;
  return (launchDarklyClient = await initialize());
}

export async function getFlagValue(key, user) {
  const client = await getClient();
  let flagValue;

  if (!user) {
    user = {
      key: "anonymous",
    };
  }
  flagValue = await client.variation(key, user, false);
  return flagValue;
}
```

Within the example code, any server-side logic that requires a flag value just imports the `getFlagValue` function and passes it the key and user data.

```javascript
import { getFlagValue } from "../../lib/launchdarkly/server";

...

const myFlag = await getFlagValue("my-flag");
```

There is no need to initialize the client as that is handled by the library as needed. Note that the `getFlagValue` function is asynchronous, so it returns a JavaScript Promise. You can use the `await` keyword if you're using it within an `async` function or use `.then()` otherwise.

### Using server-side flag values

The example code includes several routes that use LaunchDarkly flags to alter server-side logic in the application. The server-side logic that calls the Node.js SDK is kept within endpoints to ensure that the code runs only on the server.

The following example imports the server-side wrapper and uses the `getFlagValue()` function to load the value of the `featured-username` flag. This is a string flag that contains the the username of the person whose posts the home page will load. The posts and the username are passed back from the endpoint.

```javascript
import { getFlagValue } from "../../lib/launchdarkly/server";

export async function get() {
  const featuredUsername = await getFlagValue("featured-username");
  const response = await fetch(
    `https://dev.to/api/articles?username=${featuredUsername}&page=1&per_page=10`
  );
  let posts = await response.json();

  // clean up the username and organization
  posts = posts.map((post) => {
    post.username = post.organization
      ? post.organization.username
      : post.user.username;
    return post;
  });

  return {
    body: {
      posts,
      featuredUsername,
    },
  };
}
```

The data from this endpoint can be consumed in the `index.svelte` file by loading it with `fetch`.

```javascript
<script context="module">
  export async function load({ fetch }) {
    const res = await fetch("/api");

    if (res.ok) {
      let data = await res.json();
      return {
        props: { posts: data.posts, featuredUsername: data.featuredUsername },
      };
    }

    return {
      status: res.status,
      error: new Error(),
    };
  }
</script>
```

In the following example, the endpoint uses the value of a string flag to determine which version of Markdown content to load. This example could be easily modified to pass in user data to `getFlagValue`. The `new-about-us` flag could be targeted to only internal users, allowing the marketing team to test new copy in production without impacting external users.

```javascript
import { processMarkdown } from "../../lib/markdown";
import { getFlagValue } from "../../lib/launchdarkly/server";

export async function get() {
  const aboutUsPage = await getFlagValue("new-about-us");
  const { frontMatter, body } = await processMarkdown(
    `./src/posts/${aboutUsPage}.md`
  );

  return {
    body: {
      frontMatter: frontMatter,
      body: body,
    },
  };
}
```

## Client-side rendering

Svelte and SvelteKit don't require any special framework libraries to work with LaunchDarkly. You can work with the standard JavaScript client-side SDK. However, you do need to ilsolate calls to the SDK into code that only runs on the client, not the server.

### Creating a client-side wrapper

Much like the server-side interaction, it can be useful to create a wrapper for interaction with the client-side SDK. This can accomplish several things such as isolating the initialization code to a single location, automatically initializing the client if it hasn't been initialized yet and even automatically adding change listeners on flags.

The client-side wrapper is in the file `/src/lib/launchdarkly/client.js`. The wrapper does not implement all the SDK functionality but has the following functions:

* `initialize()` handles initializing the SDK using the LaunchDarkly SDK Key that is stored in an environment variable via a `.env` file. It then waits for the SDK to initialize and returns the SDK client.  If no user data is passed, the function will pass a generic anonymous user key.
* `getClient()` simply checks to see if the client has already been initialized and, if not, initializes it. In either case, it returns an instance of the SDK client.
* `getFlagValue` will return the value of a LaunchDarkly flag based upon the flag's key. If a callback function is passed, it will add that as a change listener specific to the passed tag name, meaning that this function will only be called on a change to that specific flag rather than a change to any flag.

```javascript
import * as LaunchDarkly from "launchdarkly-js-client-sdk";

let launchDarklyClient;

async function initialize(user) {
  if (!user) {
    user = {
      key: "anonymous",
    };
  }
  const client = LaunchDarkly.initialize(
    import.meta.env.VITE_LAUNCHDARKLY_SDK_CLIENT,
    user
  );
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

Within the example code, any client-side logic that requires a flag value just imports the `getFlagValue` function. You'll also need the `browser` value imported in order to perform a check within your code to ensure that it is running within the browser environment.

```javascript
import { browser } from "$app/env";
import { getFlagValue } from "../launchdarkly/client";
```

Before you can use the function, you'll need to wrap it in a `browser` check. In addition, unless the call to `getFlagValue` exists within an `async` function, you'll need to use a `.then()` to properly set the value. A simple solution for this is to create a setter for the value.

```javascript
let myflag;
if (browser) {
  getFlagValue("my-flag").then(setMyFlag);
}

function setMyFlag(val) {
  myflag = val;
}
```

There is no need to initialize the client as that is handled by the library as needed. Note that the `getFlagValue` function is asynchronous, so it returns a JavaScript Promise.

If you'd like to set a change listener for this particular flag value, you can pass the setter as a secondary argument to `getFlagValue`.

```javascript
let myflag;
if (browser) {
  getFlagValue("my-flag", setMyFlag).then(setMyFlag);
}

function setMyFlag(val) {
  myflag = val;
}
```

If a change to the flag within LaunchDarkly is detected, the setter function will be called and your application's UI will update accordingly.

### Using flag values client-side in JavaScript

The below example shows two examples of how client-side flag values can be set within a Svelte component. The first, `show-about-us`, is a boolean flag that will determine whether the "About Us" element appears within the navigation. It uses a basic setter function that is set to listen for changed to the flag on LaunchDarkly.

The second, `featured-category`, is a string flag that is used both server-side and client-side to determine which blog post category is featured. On the client-side, this impacts the featured category navigation item. This example shows that the setter can be used to massage the result. In this case, the setter title cases the returned category string.

```javascript
<script>
  import { browser } from "$app/env";
  import { getFlagValue } from "../launchdarkly/client";

  let showAboutUs;
  let featuredCategory;
  if (browser) {
    getFlagValue("show-about-us", setShowAboutUs).then(setShowAboutUs);
    getFlagValue("featured-category", setFeaturedCategory).then(
      setFeaturedCategory
    );
  }

  function setShowAboutUs(val) {
    showAboutUs = val;
  }

  function setFeaturedCategory(val) {
    featuredCategory = val.charAt(0).toUpperCase() + val.slice(1);
  }
</script>
```

### Using client-side flag values within markup

There's no need for any complex markup or even the use of an [`await` block](https://svelte.dev/tutorial/await-blocks) in the markup because the value of either flag variable above, `showAboutUs` and `featuredCategory`, never exists as a pending Promise. The values are either undefined while the SDK is initialized and the flag state is returned or it is defined with either a boolean or string value respectively. As shown in the example below, the string value can be included as needed in the markup and the boolean value can used for conditional rendering.

```html
<header>
  <div class="nav">
    <ul>
      <li>
        <a href="/">Home</a>
      </li>
      <li>
        <a href={`/posts/${featuredCategory}`}>Posts from {featuredCategory}</a>
      </li>
      {#if showAboutUs}
        <li class="about">
          <a href="/about">About Us</a>
        </li>
      {/if}
    </ul>
  </div>
</header>
```

There are two important things to note here:

1. Svelte will handle the DOM updates necessary to the rendering once the values become defined. There is no need for any JavaScript code to ensure these values are updated.
2. Both values will update immediately if a change is made to them in LaunchDarkly as change listeners have been added to them both. As soon as a new value is received, Svelte will update the UI accordingly.

## Conclusion

Svelte continues to grow in popularity, jumping from 8% usage in 2019 to 20% in 2021 while remaining the frontend framework JavaScript developers are most interested in, according to the latest [State of JavaScript survey](https://2021.stateofjs.com/en-US/libraries/front-end-frameworks). As it becomes an integral part of more companies' web site infrastructure, it becomes increasingly important that they can integrate feature management into Svelte web applications. The great news is that, because Svelte is just JavaScript, you can already do that today using LaunchDarkly without the need for any specialized SDKs.