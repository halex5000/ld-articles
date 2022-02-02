# Using LaunchDarkly with TypeScript

TypeScript's usage and popularity has exploded in recent years. According the [most recent StackOverflow survey](https://insights.stackoverflow.com/survey/2021), it is already the [seventh most popular language](https://insights.stackoverflow.com/survey/2021#technology-most-popular-technologies), ahead of languages such as C#, PHP and Go. Not only that, but it is the [second most wanted language](https://insights.stackoverflow.com/survey/2021#technology-most-loved-dreaded-and-wanted), meaning more developers want to learn TypeScript. So expect its usage to grow. As a [recent article on CSS Tricks states](https://css-tricks.com/the-relevance-of-typescript-in-2022/):

> TypeScript is not just evolving; it is exploding and only gaining momentum as we settle in 2022. So, yes, TypeScript is relevant in 2022… and will continue to be for some time to come.

If you haven't heard of [TypeScript](https://www.typescriptlang.org/), it is a superset of JavaScript, meaning that any valid JavaScript is valid TypeScript. It then layers on a whole bunch of features, most notably support for type checking. The TypeScript compiler compiles the TypeScript code to JavaScript.

If you are already using TypeScript in your application development or looking to start, this post will show you how to get LaunchDarkly working within your TypeScript code using the Node.js SDK and how to integrate the React SDK for the frontend in a TypeScript project.

## Using the Node SDK in a TypeScript Project

Since valid JavaScript is valid TypeScript, there really isn't anything specific that you need to do to use the LaunchDarkly Node.js (Server) SDK within TypeScript application code. However, you'll likely want to use type checking – that's the point, right? Fortunately, some types are provided that can help.

Let's take a look at the following example and go over what it is doing:

1. You'll need to import that `launchdarkly-node-server-sdk` of course. In addition, the example uses `dotenv` to import the SDK key from a `.env` file, which is added to our `.gitignore`.
2. The `ldClient` variable holds a copy of the LaunchDarkly client that will be returned once the SDK is initialized, which will be of the type `LaunchDarkly.LDClient`. The `getClient()` function handles the initialization using the SDK key, which is available on the [account settings](https://app.launchdarkly.com/settings/projects) in the LaunchDarkly dashboard. Once the client is initialized, `getClient()` returns it.
3. `getFlagValue()` is a generic wrapper around the client's `variation()` method to get a flag's current value. It handles initializing the client if it doesn't exist and populating an anonymous user key if one is not provided for user targeting. It passes these values to the client to return the current flag value, which is of type `LaunchDarkly.LDFlagValue`.
4. Finally, the `getUsername()` method is just some dummy code to retrieve a the flag value for a string flag named `featured-username` and log it to the console. Flag values can be booleans, string or objects (for JSON flags).

```typescript
import * as LaunchDarkly from "launchdarkly-node-server-sdk";
import * as dotenv from "dotenv";
dotenv.config({ path: __dirname + "/.env" });

let ldClient: LaunchDarkly.LDClient;

async function getClient(): Promise<LaunchDarkly.LDClient> {
  const client = LaunchDarkly.init(process.env.LAUNCHDARKLY_SDK_KEY);
  await client.waitForInitialization();
  return client;
}

async function getFlagValue(
  key: string,
  user: LaunchDarkly.LDUser | null,
  defaultValue: any = false
): Promise<LaunchDarkly.LDFlagValue> {
  let flagValue: LaunchDarkly.LDFlagValue;
  if (!ldClient) ldClient = await getClient();
  if (!user) {
    user = {
      key: "anonymous",
    };
  }
  flagValue = await ldClient.variation(key, user, defaultValue);
  return flagValue;
}

async function getUsername(): Promise<any> {
  const username: string = await getFlagValue("featured-username", null);
  console.log(username);
}

getUsername();
```

While this is a pretty basic example, you'll see that it takes advantage of the types made available by the SDK to incorporate type checking throughout. You can find all the types available in the Node SDK via the [API documentation](https://launchdarkly.github.io/node-server-sdk/modules/_launchdarkly_node_server_sdk_.html). You can also clone this example code [on GitHub](https://github.com/launchdarkly-labs/ld-basic-typescript).

## Using the React SDK in a TypeScript React Project

If you try to use the React SDK within a React application that uses TypeScript, you may encounter some compiler errors or warnings initially. Fortunately, these are relatively easy to fix and, once fixed, you can just use the SDK [as documented](https://docs.launchdarkly.com/sdk/client-side/react/react-web).

Let's look at an example that uses Next.js via their [blog-starter-typescript](https://github.com/vercel/next.js/tree/canary/examples/blog-starter-typescript) template. The key changes that are required are in the `/pages/_app.tsx` that creates the root application component. Here is the original code.

```javascript
import { AppProps } from 'next/app'
import '../styles/index.css'

export default function MyApp({ Component, pageProps }: AppProps) {
  return <Component {...pageProps} />
}
```

A standard solution for React applications using LaunchDarkly's React SDK would be to wrap the `MyApp` component in a `withLDProvider` function.

```javascript
import { AppProps } from 'next/app'
import '../styles/index.css'

function MyApp({ Component, pageProps }: AppProps) {
  return <Component {...pageProps} />
}

export default withLDProvider({
  clientSideID: process.env.LAUNCHDARKLY_SDK_CLIENT,
})(MyApp);
```

Doing so will bring up two TypeScript compiler errors.

1. `clientSideID` will report that `Type 'string | undefined' is not assignable to type 'string'`
2. Passing `MyApp` to `withLDProvider` will result in `Argument of type '({ Component, pageProps }: AppProps<{}>) => Element' is not assignable to parameter of type 'ComponentType<{}>'.`

The code below resolves these issues first by adding a `!` to `process.env.LAUNCHDARKLY_SDK_CLIENT` to indicate that it will not be undefined. Second, `MyApp` is coerced to type `ComponentType<{}>`.

```typescript
import { AppProps } from 'next/app'
import '../styles/index.css'
import { withLDProvider } from "launchdarkly-react-client-sdk";

function MyApp({ Component, pageProps }: AppProps) {
  return <Component {...pageProps} />
}

export default withLDProvider({
  clientSideID: process.env.LAUNCHDARKLY_SDK_CLIENT!,
})(MyApp as ComponentType<{}>);
```

That's it. Now the application can use SDK toolks like `withLDConsumer` or the `useFlags` hook to get client-side flag values within the frontend of the application.

## Flags + Types = 🙌

As we've seen, it's easy to incorporate feature flags into your TypeScript application. We explored how to do this using the Node SDK on the server or the React SDK on the client within a full stack application built with TypeScript. If you'd like to dig further into what's available within each SDK, be sure to check the [Node.js SDK (server-side) documentation](https://docs.launchdarkly.com/sdk/server-side/node-js) or the [React SDK documentation](https://docs.launchdarkly.com/sdk/client-side/react/react-web).