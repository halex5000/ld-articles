# 5 Things That Make Astro Unique for Building Web Apps

It can sometimes feel like there's a new framework or developer tool every week, so you can be forgiven for wanting to roll your eyes at yet another tool. But occassionally one of these tools seems to really shift the paradigm of how we develop applications. [Astro](https://astro.build/) just might be one of those tools when it comes to building web applications.

Astro, which just released it's [1.0 beta](https://astro.build/blog/astro-1-beta-release/) last month, is a static site generator (SSG) like Gatsby, Next.js or Nuxt. Like those tools, the result can be far from static, as Astro even supports server-side rendering (SSR) of any route within the application and even [rendering at the edge](https://astro.build/blog/netlify-edge-functions/). However, Astro differs from those tools in a number of ways that, I believe, makes it uniquely innovative. In this post, I'll explore what those innovations are and even how you can integrate LaunchDarkly within an Astro application.

## Less client-side JavaScript

Many SSGs are built upon the foundation of a particular JavaScript framework like React or Vue. This means that the compiled application that the SSG creates bundles the framework. This is true even for pages that are statically rendered and may have no specific need for the framework.

Astro, similar to other tools like Eleventy, generates zero client-side JavaScript by default. You can still use components and imports to modularize your application, but Astro optimizes the output to eliminate client-side JavaScript wherever it isn't needed. So, your "About Us" static page that doesn't contain any client-side interactivity will also not contain any JavaScript.

Astro claims that this optimization allows the tool to ship 90% less JavaScript than other SSGs. Less client-side JavaScript doesn't just mean a smaller bundle for your users to download, it can also mean better performance in the browser.

## Use any framework - or none

Shipping less JavaScript doesn't mean that you cannot use your JavaScript framework of choice. While Astro supports it's own Astro components format, you can also write components using React, Vue, Svelte and more. You can even mix and match frameworks if you really want to – even rendering components from multiple frameworks in the same Astro component (as in the example below from the Astro documentation).

```javascript
---
import MyReactComponent from '../components/MyReactComponent.jsx';
import MySvelteComponent from '../components/MySvelteComponent.svelte';
import MyVueComponent from '../components/MyVueComponent.vue';
---
<div>
  <MySvelteComponent />
  <MyReactComponent />
  <MyVueComponent />
</div>
```

By default the component will render as static HTML without client-side JavaScript, but what if your component needs client-side JavaScript to render?

## Partial hydration

Astro has tools built in to allow you to selectively make components interactive (referred to as hydration). It provides [special directives](https://docs.astro.build/en/core-concepts/framework-components/#hydrating-interactive-components) to tell Astro when a component requires client-side JavaScript. This lets you tell Astro that the contents of a particular component should be rendered in the browser rather than during the build.

For example, in the following example from the documentation, the first React component will render on the client immediately on page load, while the second will render only when the user scrolls and the component becomes visible.

```javascript
---
import InteractiveButton from '../components/InteractiveButton.jsx';
import InteractiveCounter from '../components/InteractiveCounter.jsx';
---
<!-- This component's JS will begin importing when the page loads -->
<InteractiveButton client:load />

<!-- This component's JS will not be sent to the client until 
the user scrolls down and the component is visible on the page -->
<InteractiveCounter client:visible />
```

This is referred to as partial hydration because Astro will embed the client-side JavaScript necessary for that component while still rendering the rest of the page statically. This architecture is commonly called the [islands architecture](https://jasonformat.com/islands-architecture/) because one component (or widget) on the page may contain JavaScript while others do not.

In addition, components are loaded individually and in isolation, so that the rendering of one component is not blocked by the rendering of another while performance issues with one also don't affect the performance of another.

## Markdown


Most, though not all, SSGs have some built-in mechanism for rendering and displaying Markdown content as HTML. Astro takes this a step further though and allows you to do some very dynamic things with Markdown that are unique.

For example, you can use variables defined in the frontmatter (i.e. the metadata at the top of the Markdown file) within the Markdown content.

```markdown
---
answer: 42
---
The meaning of life is {frontmatter.answer}
```

You can import components in an Astro file and then use them within the rendered Markdown content. These can even include framework-based components that rely upon partial hydration to render.

```html
---
setup: |
  import Author from '../../components/Author.astro'
author: Brian
---

<Author name={frontmatter.author}/>
```

Finally, you can easily  import single Markdown files or even an entire directory of Markdown files and use them as data as in this example from the documentation:

```markdown
---
import * as greatPost from '../pages/post/great-post.md';
const posts = await Astro.glob('../pages/post/*.md');
---

Great post: <a href={greatPost.url}>{greatPost.frontmatter.title}</a>

<ul>
  {posts.map(post => <li>{post.frontmatter.title}</li>)}
</ul>
```

It's also worth noting that Astro includes a `<Markdown/>` component for rendering Markdown as well as multiple built-in components for rendering code blocks both server-side or client-side.

## No configuration static assets

Astro makes it easy to include any number of static assets in your build and then simply `import` them into any component or page as needed. This doesn't just include importing other components like Astro components (`.astro`)  or React components (`.jsx`) or Markdown as discussed above.

You can also import JavaScript, TypeScript, JSON, npm packages, CSS and image assets with no configuration required. This makes integrating almost any kind of static asset easy while taking advantage of Astro's built in bundling and optimizations. 

## Using LaunchDarkly within an Astro project

If you're looking to combine both LaunchDarkly and Astro together, I've got good news. The existing [Node.js SDK](https://docs.launchdarkly.com/sdk/server-side/node-js) and [client-side JavaScript SDK](https://docs.launchdarkly.com/sdk/client-side/javascript) work out-of-the-box without any modifications. Let's quickly see how to integrate each.

### Build-time or server-side

If you would like to implement the Node.js SDK, you can simply import it, initialize the client and then start getting flag variations without any special code. For example, you could include the following code on any page or component to use a flag within the code – yes, even in the frontmatter of a Markdown file.

```javascript
---
import LaunchDarkly from 'launchdarkly-node-server-sdk'
const client = LaunchDarkly.init(import.meta.env.LAUNCHDARKLY_SDK_KEY);
await client.waitForInitialization()
const myFlag = await client.variation('featured-category', {key: "anonymous"}, false);
---

<p>{myFlag}</p>
```

However, you probably don't want to initialize the connection to LaunchDarkly on every single page that uses a server-side flag. You can utilize a strategy I discussed in my [Wrapping LaunchDarkly](https://launchdarkly.com/blog/wrapping-launchdarkly/) article to initialize the server-side SDK once by creating a `ld-server.js` file as follows:

```javascript
import LaunchDarkly from "launchdarkly-node-server-sdk";
const client = LaunchDarkly.init(import.meta.env.LAUNCHDARKLY_SDK_KEY);

export default async function () {
  await client.waitForInitialization();
  return client;
}
```

You can then pass this to a [LaunchDarkly user wrapper](https://github.com/remotesynth/ld-astro/blob/main/src/components/LaunchDarkly/ld-user.js) and then get flag variations.

```javascript
---
// launchdarkly libraries
import ldServer from "../components/LaunchDarkly/ld-server";
import ldUser from "../components/LaunchDarkly/ld-User";

let client = await ldServer();

// each user would be initialized with their info
let user = new ldUser(client);
const myFlag = await user.getFlagValue("featured-category");
---

<p>{myFlag}</p>
```

Keep in mind that if a flag impacts a build-time rendering, you'll likely want to trigger a rebuild when the flag has changed using LaunchDarkly's [webhooks integration](https://docs.launchdarkly.com/home/connecting/webhooks). You can read more about how this is done with examples for both Netlify and Vercel in my [Next.js guide](https://docs.launchdarkly.com/guides/platform-specific/nextjs#considering-build-impact). This is not necessary for server-side rendering (SSR).

### Client-side

No special, Astro-specific code is needed to use LaunchDarkly client-side within an Astro application. For example, if you wanted to use the client-side SDK in an Astro component, you can just wrap the standard initialization and flag variation calls in a `<script>` block.

```javascript
<script>
  import * as LDClient from 'launchdarkly-js-client-sdk';

  const client = LDClient.initialize(import.meta.env.PUBLIC_LAUNCHDARKLY_CLIENT_ID, {key:"anonymous"});
  await client.waitForInitialization();
  setClientFlag(client.variation('show-feature', false));
  client.on("change:show-feature", setClientFlag);

  function setClientFlag(val) {
    alert("Flag is: " + val);
  }
</script>
```

If you are including it within a framework component, be sure to use the directive to enable hydration on that component.

As with the server-side example, you may also prefer to create a [wrapper for the JavaScript client-sdk](https://github.com/remotesynth/ld-astro/blob/main/src/components/LaunchDarkly/ld-client.js) and use that instead, which can make it easier to use.

```javascript
<script>
	import { getFlagValue } from "../lib/ld-client";
	getFlagValue("show-feature", setClientFlag).then(setClientFlag);

	function setClientFlag(val) {
		alert("Flag is: " + val);
	}
</script>
```

## Take Off with Astro and LaunchDarkly

While it is definitely still early days with Astro, it clearly brings a ton of innovations into the process of building web site and web applications. Plus, you can leverage LaunchDarkly without needing to manage some complex integration.

You can find all of the LaunchDarkly integration examples listed above, plus the full source of the SDK wrappers discussed in [this GitHub repository](https://github.com/remotesynth/ld-astro). The sample project adds some LaunchDarkly flags into the basic blog example from [astro.new](https://astro.new).



