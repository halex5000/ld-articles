# What's So Great About Next.js?

If you're working in web development today, chances are that you're are aware of, curious about or perhaps even used [Next.js](https://nextjs.org/). Next.js is what's often referred to as a "meta framework" in that it is a framework built upon one or more other frameworks. In the case of Next.js, it is built upon React.

As React became the most widely adopted web framework, encompassing over 40% of developers in 2021 [according to Statista](https://www.statista.com/statistics/1124699/worldwide-developer-survey-most-used-frameworks-web/), Next.js adoption has also grown dramatically. Vercel, the company that maintains Next.js, [cites](https://venturebeat.com/2021/06/23/vercel-secures-102m-to-accelerate-next-js-adoption/) over 6 million downloads since its launch and 50% usage growth in the top ten-thousand websites for the period of October 2020 to June 2021 alone.

A big part of the reason that developers and companies are choosing Next.js for their web application development is because it is a full-stack framework (as in, it handles both the frontend and backend of your application) and offers support for a variety of rendering methods – even mixing and matching those methods as needed. Let's take a brief look at how Next.js has evolved and, in particular, how the rendering methods within Next.js have evolved.

> Want a more detailed exploration of how to combine Next.js and LaunchDarkly feature management? Check out our guide [Using LaunchDarkly with Next.js](https://docs.launchdarkly.com/guides/platform-specific/nextjs).

## The Evolution of Next.js

Today, Next.js is frequently associated with the [Jamstack](https://jamstack.org) methodology for developing web application, but, when it was launched in early 2016, it was [originally released](https://vercel.com/blog/next) it was just for server-side rendered apps. What made Next.js unique at the time was that it handled both the frontend and the backend of your application and both were built using JavaScript and React.

Next.js also gained popularity because it simplified building full-stack applications using React. It provided automatic routing for pages and built-in methods for server-side rendering (SSR) and data fetching. All of these features still exist in Next.js today but many new features have been added and the existing ones have been dramatically enhanced. In particular, Next.js now handles multiple types of rendering methods.

### Rendering Methods in Next.js

As we noted, Next.js started as simply a framework for SSR but as that has expanded, even what SSR means in Next.js has changed. Let's look at the various rendering options available in Next.js today:

**Server-side rendering (SSR)**
In SSR, content is generated on the server (which is Node.js) for every single request and then sent to the browser. Startinh with release of [Next.js 8](https://nextjs.org/blog/next-8), every server-rendered page became a serverless function (also known as a lambda). For instance, when we call the page at `/about`, Next.js calls a serverless function that specifically handles returning the backend data necessary to render the "About" page. The data fetching is encapsulated in the `getServerSideProps()` method in Next.js.

**Pre-rendering**
With pre-rendering – also called static rendering or static site generation (SSG) – the page is rendered during a build that occurs before the application is deployed, usually as part of a CI/CD build process. This was originally added as an option in [Next.js 3](https://auth0.com/blog/nextjs-3-release-what-is-new/) but the process was very manual. As of  [Next.js 6](https://nextjs.org/blog/next-6), routes became automatic for both SSR and SSG pages (though dynamic routes still need to provide paths programmatically via the `getStaticPaths()` method). [Next.js 9](https://nextjs.org/blog/next-9) introduced a feature called "Automatic Static Optimization" that automatically determines if a page can be rendered as static. The ability to mix server-side rendered pages/routes and pre-rendered pages/routes was unique to Next.js and has since been adopted by other tools frameworks like Gatsby and Nuxt.

**Deferred rendering (ISR)**
Within Next.js, deferred rendering is referred to as Incremental Static Regeneration (ISR), which was initially introduced in [version 9.4](https://nextjs.org/blog/next-9-4).  It is similar to pre-rendering, though the requested page isn't rendered during the initial build but instead when it is first requested by a user. Subsequent users will see the prerendered version of the page either until a new build occurs or until an optional cache timeout has passed. The goal of ISR is to eliminate the extremely long build time that large sites could often face by allowing the developer to defer building lower-trafficked pages. It also has the ability to be selectively used to render pages based upon user-generated content.

Of course, as with any frontend framework, Next.js also has methods to assist with **client-side rendering**, where content can be loaded, modified or updated via client-side JavaScript.

## With Great Power...

The ability to mix and match all of these rendering methods gives developers a lot of power, but it also presents a unique challenge. Developers now have to consider more than just how to render specific content, but _when_ to render it. While by no means comprehensive, here are a few things to consider:

* Is this content the same for every user? Then pre-rendering or deferred rendering can offer the best performance.
* Is this content user-specific or generated dynamically for each request? Then server-side rendering may be the best option, but, in some cases, deferred rendering may be able to achieve similar results.
* Is this content user/request-specific but lightweight or does it require real-time updates to the page? Then this can probably be loaded via client-side rendering.

This also becomes complicated when integrating a tool like LaunchDarkly. Which SDK(s) should I use? How do I integrate the SDKs into Next.js code? How does LaunchDarkly work with pre-rendered or deferred rendered pages? We answer all of these questions and more in our latest guide, [Using LaunchDarkly with Next.js](https://docs.launchdarkly.com/guides/platform-specific/nextjs).

