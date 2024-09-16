# What Are Feature Flags?

Feature flags, or feature toggles as they are often called, allow you to enable or disable a feature without modifying the code or requiring a redeploy, instead changes are made by determining at runtime which portions of code are executed. This allows new features to be deployed without making them visible to users or, even more powerfully, you can make them visible to only specific users.

At the most basic level, they take the form of simple conditionals within the code.

```javascript
if (featureFlag === true) {
  // show new feature
}
else {
  // show old feature
}
```

The flag is typically managed remotely, meaning that the feature can be turned on or off even as the underlying code remains the same. (It's worth quickly noting here that ultimately feature flags do [not need to be limited to just boolean values](https://launchdarkly.com/blog/feature-flags-beyond-the-boolean/).)

By using feature flags, a feature that is either unfinished or not fully ready for public consumption can exist in the application's code, but it will not impact users or potentially break the application. It also means that a feature that has been released but is malfunctioning can be turned off without requiring a full rollback to a previous version of the application. Essentially, feature flags decouple deployment from release – just because a feature is deployed in the code does not mean it is released.

## Decoupling Deploy from Release

Historically, deploy and release were synonymous because code that was deployed to production was automatically running in production. This meant that every new deploy carried significant risks to the existing application. As a way to alleviate these risks, teams would bundle multiple features into releases that might happen once a week, once a month or even once a quarter. However, this actually raised the stakes of a deploy significantly and, if any single feature was broken in production, the entire bundled release needed to be rolled back, dramatically slowing down the velocity of development teams.

According to the *[Accelerate: State of DevOps Report](https://cloud.google.com/blog/products/devops-sre/the-2019-accelerate-state-of-devops-elite-performance-productivity-and-scaling)*, one of the differentiating characteristics of high performing organizations compared to low performing teams is that they deploy code 46 times more frequently, with a lead time from commit to deploy that is 2,555 times faster. Although they deploy code _more_ frequently, they also fail _less_ frequently. High-performing teams have a lower change failure rate and recover from incidents faster. Part of what makes this possible is having the right tools, systems and processes in place.

Feature flags change the traditional deployment process by decoupling deploy and release, allowing new code to exist in a production deploy but not be executed and therefore not released. This can dramatically change the way development teams operate for multiple reasons:

* **Trunk based development** – There is no longer a need for maintaining multiple long-lived feature branches. These are frequently the cause of complex merge conflicts that can result in difficult to debug issues. Trunk-based development is a practice in which the development teams all work in the main branch and/or frequently merge short lived branches. This not only removes the complexity of merges, but also enables teams to move faster by allowing them to deploy changes continuously.
* **Manage the feature lifecycle** – Controlling who has access to a feature enables teams to control the full lifecycle of that feature. For example, a feature flag can make new features available only to internal users for testing and feedback before being released publicly. Or a release can be timed to align to align with a marketing campaign without requiring a deploy or even intervention by engineering at all. Simply flip the flag and the feature is live with the full confidence that the success of the campaign will not be undermined by a failed or delayed deployment.
* **Remove the stress and risk around releases** – Feature flags reduce the stress around releases by allowing faster resolution of incidents (i.e. mean time to remediate, which is a key DORA metric). If a feature is causing problems, flipping the flag off will disable it, solving the problem immediately without the need to roll back a complex release that may contain multiple critical features or fixes. Feature flags even enable [Progressive Delivery](https://launchdarkly.com/blog/what-is-progressive-delivery-all-about/) whereby a feature is gradually released to larger populations of users to ensure that it performs as expected, which can enable teams to detect problems earlier and reduce the impact of any issues.

### Who uses feature flags?

Since feature flags are implemented in code, they require a development or engineering team to configure them within the application. As a result these teams are typically the first to adopt feature flags for managing feature releases. However, that does not mean that their utility is limited to development or engineering teams.

Feature flags are the underpinning of a [feature management](https://launchdarkly.com/feature-management/) solution, which enables teams across the company to perform tasks that previously required engineering intervention. For example, utilizing a feature management platform with feature flags can enable:

-   QA to test in production and validate changes.
-   Sales and Customer Support to provision entitlements.
-   Product Management to manage beta program access or align releases with non-engineering business priorities.
-   Marketing to run A/B tests or time releases to campaigns.

Pairing feature flags with a feature management platform makes toggling flags easy to understand can enable release stakeholders across the company to contribute to the release lifecycle.

### When to use feature flags

There are four main categories for when to use feature flags.

#### Improve feature rollout (release management)

Using flags as part of your build or feature release process is one of the most common uses of feature flags. [Release management](https://launchdarkly.com/blog/release-management-flags-best-practices/) includes early access programs, canary releases, and beta programs---anywhere you give specific users access to features before releasing the feature to everyone. Starting small and rolling out to larger groups over time helps you:

-   Observe the behavior of the systems and services under increasing load.
-   Collect user feedback and make changes if you need to.
-   Limit the blast radius if something goes wrong

![When to use feature flags diagram](https://images.prismic.io/launchdarkly/d0b2f047-8b42-4bd6-9ccd-41abe680728f_when-to-use-feature-flags.png?auto=compress,format)

#### Operational efficiency

Feature flags can manage and control the behavior of a system toggling a feature on/off to minimize the impact of incidents. Using [operational feature flags](https://launchdarkly.com/blog/operational-flags-best-practices/) you can:

-   Deploy circuit breakers or kill switches to programmatically or manually disable a feature if it negatively impacts the user experience and alerts are triggered.
-   Limit API requests to ensure API reliability.
-   Switch to a less feature-rich version of a page under heavy load.
-   Test new microservices or third-party tags in production for interoperability testing.
-   Change log-levels on the fly for debugging purposes.

#### Learning through experimentation

Monitoring and observability tools can tell you if a feature isn't working correctly, but they can't tell you if you if you built the right feature. [Experimentation](https://launchdarkly.com/blog/nine-experimentation-best-practices/) or A/B testing can offer insights into things like how users are using your feature and/or whether it is performing as intended to ensure you release the correct feature. 

Using feature flags, you can test different configurations of a feature to validate or disprove hypotheses. Experiments provide concrete reporting and real world measurements to ensure that you are launching the best version of a feature that positively impacts company metrics.

![Learn from Experimentation diagram](https://images.prismic.io/launchdarkly/169abf00-f66d-4837-9d79-97c9559fab1e_learn-from-experimentation.png?auto=compress,format)

#### Empower teams with entitlements

In software, *entitlement* refers to the right to use services, products, or features. Entitlements define the features each user can access.

Feature flags, in the context of a feature management platform, can empower others in the organization to manage [and control entitlements](https://launchdarkly.com/blog/how-to-manage-entitlements-with-feature-flags/), even [within a multi-tenant SaaS environment](https://www.youtube.com/watch?v=uzRrEWzqD0Y). This results in better customer experiences and greater operational efficiency.

![Empower teams with entitlements image](https://images.prismic.io/launchdarkly/a6e31483-d245-4c89-ae26-36f4fad2d315_empower-teams-with-entitlements.png?auto=compress,format)

### How are feature flags deployed?

How you choose to deploy feature flags depends on several factors including:

-   The size of your engineering team
-   How often you release code to production
-   How many flags you want to deploy
-   Whether you want to [build vs. buy](https://launchdarkly.com/blog/feature-management-platform-build-or-buy/)

Many companies start using feature flags via a configuration file, often created in a markup language like YAML, or configuration values in a service like AWS AppConfig. This type of homegrown solution works well for flags that are either "on" or "off" for all users. However, it does require developer intervention to change a flag and, in the case of a physical config file, it may require a redeploy of changes. Also, because using config files can get too long, it tends to be a better solution for project that only want a small number of flags.

For a large number of flags, or for more complex homegrown solutions that may include a management dashboard, you'll likely opt to store flags in a database. Depending on the complexity of your solution, this method can allow features to be targeted to individual users. You also can make modifications to user targeting without having to restart or re-deploy services. However, any management tools or targeting logic would need to be created and maintained in-house. In addition, the solution may lack the ability to push changes back down to the application so that, for instance, if a kill switch is flipped, the effect is immediate.

There are some open-source feature flag solutions available as well as a vendor neutral solution called [OpenFeature](https://featureflags.io/resources) from the CNCF. Most open source solutions typically allow for user segmentation, targeting, and controlled feature rollouts and include some form of management dashboard, but some solutions may be language or framework specific, which may not allow for flag usage across projects that include multiple languages.

A final deployment option is to use an enterprise feature management solution like LaunchDarkly. When you have needs such as role-based access controls (RBAC), audit logs, streaming updates, or use cases including experimentation or entitlements, you would want to deploy a comprehensive feature management solution.

### Why use LaunchDarkly for feature flags?

LaunchDarkly offers a [feature management platform as a service](https://launchdarkly.com/product/) designed for the entire organization. We believe the effective use of feature flags is an organization-wide endeavor, not something limited to a few developers. All teams across the organization can benefit from the use of feature flags. LaunchDarkly grows with you as your feature flagging practice expands from a single flag deployed by a single developer to team-wide use, to delegating control of flags to other teams to meet their use cases.

[Request a demo](https://launchdarkly.com/request-a-demo/) of the LaunchDarkly feature management platform or [start a free trial](https://launchdarkly.com/start-trial/).
