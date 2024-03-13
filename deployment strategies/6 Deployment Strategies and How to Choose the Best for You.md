# 6 Deployment Strategies and How to Choose the Best for You

Traditional deployment strategies all involved load balancers and diverting traffic (or not). They ranged from the "big bang deployment", where you just push everything live and cross your fingers to safer alternatives like incremental deployment, canary deployments or "blue/green" deployments. The commonality these all shared was that the full release was deployed and then traffic was redirected _at the hardware level_.

The problem with these traditional strategies is that infrastructure can be expensive. It's not just the costs of the physical or (more likely) cloud infrastructure itself, but also the time and effort required to set up and maintain it.

Thankfully, feature flags and feature management systems have altered the entire equation. There's no longer a need to stand up duplicate infrastructure to do things like incremental rollouts or A/B testing. Let's explore six deployment strategies as well as the impact of using feature flags can change your approach from hardware-centric to software or code-centric.

## Differences between Deployments and Releases

The big change in how we think about and manage deployments came as we've been able to decouple deployment from release. Let's briefly explore what deploy and release traditionally meant and how those definitions have changed.

In the original concept of DevOps, release came before deploy. A release consisted of freezing code, after it had undergone testing, and marking it ready for deployment to production. The deployment was the process of moving this code to a production environment, thereby making it live.

The introduction of feature flags changed this. In a modern DevOps process, deployment now comes before release. This is because feature flags decouple the deploy from the release. This is because, even though code is deployed to production, the code may not be running. Strategies like [trunk-based development](https://launchdarkly.com/blog/introduction-to-trunk-based-development/), whereby code is constantly merged into the main branch, combined with continuous integration and continuous delivery (CI/CD) mean that new code is constantly being pushed to a production environment. However, it is only when that feature flag is enabled that it is actually released.

This also means that release no longer encompasses the entire application code, but can be thought of on a feature-by-feature basis.

## Types of Deployment Strategies

### 1. Big Bang Deployment

As the name suggests, this deployment strategy involves deploying all changes at once, impacting all users simultaneously. Traditionally, this approach was simple and fast to implement. It usually meant that the application was down during the course of the deployment (often done at off peak hours) and wasn't back up until the deployment was complete. It also carried risks because any issues in the deployment affect all users immediately, and rollback was often complex and problematic.

While big bang deployments are still not advisable, feature flags and feature management can allow you to mitigate the risks of this type of deployment by implementing a kill switch that can immediately reverse the release should an issue or incident arise. This would be functionally the same as rolling back the entire release, but without the effort and downtime that rollbacks typically require.

## Rolling (aka Incremental or Ramped) Deployments

Rolling deployments adopt a gradual rollout approach, where changes are exposed to an increasing percentage of users gradually until fully released. This strategy reduces the risk associated with big bang deployments, as any issues impact only a subset of users at any given time. They also offer easier rollback as the prior version is still running until the deployment reaches 100%, meaning that, at any point, addressing a critical issue or incident is just a matter of reverting traffic to the prior version rather than a rollback or redeploy.

The downside, traditionally, was that incremental deployments were difficult to manage and required routing a percentage of traffic at the hardware level to the updated application and ensuring that this change was sticky (i.e. the user would continue to see the same version rather than switch between versions on subsequent requests).

Feature flags alter the equation when it comes to rolling or incremental releases because they become relatively trivial to manage. You can even automate much of the process. For example, using LaunchDarkly, you could create a workflow automation using the built in [progressive rollout template](https://docs.launchdarkly.com/home/feature-workflows/workflows#building-a-progressive-rollout-workflow) and, using an integration with your observability platform, automatically kill the release should any incident or performance degredation is detected.

### 3. Recreate Deployments

Unlike the big band deployment, which overwrites the existing deployment on a running environment, recreate deployments involve terminating the existing deployment and recreating the entire environment with the newly deployed application. As you might imagine, this requires some downtime, so recreate deployments are often recommended for non-production environments (dev, QA, etc.). This approach offers simplicity and predictability is since it eliminates the need to manage complex updates to the environment but the downtime and complex rollback are significant drawbacks.

There are a number of other deployment strategies that offer the ability for deploying and even rebuilding environments (see canary or blue/green deployments) with no downtime, especially when integrating feature flags. Therefore, recreate deployments may only be suitable for specialized situations where you are only able to have a single version of the app running at one time. In these cases it may be necessary for the existing version to be completely shut down before a new deployment is possible.

### 4. Canary Deployments

The name canary deployments is based on coal miners who would use canaries to detect the presence of carbon monoxide in a mine without endangering the miners. Similarly, canary deployments are designed to target a subset of users, allowing for the early detection of issues without impacting the entire user base. Besides early detection of bugs, this process usually targets a specific subset of "power users" who can provide user feedback on changes.

Many readers may be familiar with this concept via browser makers who typically provide bleeding edge versions of their browser software via canary releases. In the case of deployments, rather than an installed canary version of the application, a user would be directed to the new version of the deployed application.

A canary deployment requires targeting a version of the application to a specific subset of users, whether they opt-in to the new version or preselected based upon specific criteria. Feature management platforms like LaunchDarkly make the complex task of managing the targeted canary segment simple. They also make it easy to roll out the new release to the full audience once it has passed the canary testing.

### 5. Blue/Green Deployments

A blue/Green deployment is one in which the existing application (blue) and the new deployment (green) are independently running in two identical production environments. Once testing is complete on the green environment, live traffic is redirected to it making it live for all users. Maintaining two environments simultaneously means that there is no downtime and it allows for for seamless rollbacks should an incident occur. One of the downsides is the costs of maintaining duplicates sets of infrastructure for the deployments. In fact, in many cases some resources like databases or other shared services are not duplicated, but shared by both the blue and green environments.

Part of the benefit of a blue/green deployment is that it allows testing in production. This eliminates potential issues caused by differences between development, Q&A and production environments. Feature flags also promote a [test in production](https://launchdarkly.com/blog/testing-in-production-for-safety-and-sanity/) strategy, although using feature flags it is possible to replicate a blue/green deployment without requiring duplicate environments. The green deployment code would be deployed to the standard production environment but hidden behind a feature flag. The new functionality would only be visible to the internal testing team and, once the release has been validated, the flag would be turned on for all users.

## 6. Shadow Deployments

At first glance, shadow deployments seem extremely similar to blue/green deployments. Both involve deploying the new release to a parallel environment without impacting end-users. Where shadow deployments differ is that they can replicate real traffic patterns under real load by replicating actual requests and sending them to the shadow environment. The output of these requests are discarded and never shown to the user (thus the name shadow) but can help uncover any issues that might occur in a real world production environment.

While shadow deployments have the inherent complexity of managing duplicate production environments and add the additional complication of duplicating requests, they are the only strategy that allows for full testing under load in a production environment without negatively impacting end users. This is a form of testing that cannot be replicated using other strategies, even using feature flags, so, depending on your circumstances, the additional overhead and complexity may be warranted.

## Choosing the Right Deployment Strategy

When choosing a deployment strategy for your application, it's worth taking into account various factors, including application characteristics, team capabilities, and your deployment goals.

- **Application Characteristics:**
  - **Size and Complexity:** It may seem obvious but simpler applications with minimal dependencies may benefit from simpler deployment strategies as they typically have lower risk of failure. More complex applications may require the sort of full production environment testing that blue/green, canary or shadow deployments allow.
  - **Downtime Tolerance:** No one likes downtime, but the tolerance for downtime can differ drastically across different applications. For example, extended downtime for an e-commerce application can be costly, while that may not be true for an internal application. In the former case you'd choose a strategy that aims for zero downtime.
  - **Risk Tolerance:** Similarly, risk tolerance can differ from application to application. In many cases, an application with low downtime tolerance will have low risk tolerance or vice versa. If your risk tolerance is low, choosing a strategy that allows for rapid or immediate mitigation of incidents (blue/green or canary) and/or that limit the impact on users (incremental or canary) is the right way to go.

- **Team Capabilities:**
  - **Skills and Experience:** As you probably noticed, some of these deployment strategies can get pretty complex and may require specialized skills and experience. A complex strategy might increase your risk if it overtaxes your team's capabilities.
  - **Resources:** It's important to evaluate whether you have the people or financial resources to manage certain strategies. Complex strategies that include duplicate environments, for instance, will naturally require more manpower to create and maintain and will also consume more financial resources in terms of the  infrastructure required.

- **Deployment Goals:**
  - **Release Frequency:** Some of these strategies are better attuned to a continuous integration / continuous deployment (CI/CD) release strategy (or as we like to call it, [operating continuously](https://launchdarkly.com/blog/modern-devops-operating-continuously/)) wherein code gets pushed to production frequently. Feature flags can help a lot here too.
  - **Testing Approach:** Strategies like canary, blue/green and shadow deployments allow for more extensive testing in a production environment and even, in some cases under production load but this testing requires additional time, resources and expertise that your team would require.
  - **User Impact:** No one likes bugs or incidents caused by a release but, let's be honest, not every application is mission critical. In some cases the cost and complexity of adopting some of the strategies might outweigh the impact of incidents. Not to sound like a broken record, but feature flags offer a low cost and complexity way of reducing user impact, even in these cases.

## Comparison of Deployment Strategies

| Strategy               | Description                                                  | Benefits                                                     | Drawbacks                                                    | When to Use                                                  |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Big Bang Deployment    | Simultaneous deployment of all changes, impacting all users at once. | Simple and fast implementation                               | High risk, no easy rollback, downtime for entire application | For small, non-critical applications                         |
| Rolling Deployments    | Gradual rollout of changes to smaller user groups, reducing risk and allowing for easier rollback options. | Lower risk, easier rollback, less downtime compared to big bang or recreate | More complex to manage, higher potential for user impact than canary/blue-green | When release frequency is a high priority                    |
| Recreate Deployments   | Recreation of the entire environment with new changes.       | Simplicity, predictability                                   | Downtime, limited testing, scalability limitations           | When there is a limitation of single version of the app running at one time |
| Canary Deployments     | Targeting a subset of users with new changes, allowing for early issue detection and real user feedback. | Early issue detection, real user feedback, easy rollback     | Slower release process than blue/green, increased infrastructure costs | Real user feedback is a priority                             |
| Blue/Green Deployments | Maintenance of two identical production environments for seamless rollbacks and improved incident response. | Easy rollbacks, improved incident response, zero downtime    | Additional infrastructure, not suitable for user-dependent   | When release frequency is a priority but there is a low risk tolerance |
| Shadow Deployments     | Deploying changes to a parallel environment without impacting users, providing low-risk testing. | Low-risk testing, with real-world data                       | Complex setup, monitoring overhead, not suitable for all changes | When testing under load is a priority but downtime tolerance is low |

## The Role of Feature Management

As we've seen throughout the discussion of the various approaches, feature management can play a critical role in most of these strategies. In some cases, adding feature management and targeting capabilities can simplify what might otherwise be a complex strategy – for example, in incremental deployments or canaries. In just about every strategy, adding a layer of feature management can help reduce the downtime risk and increase the speed of mitigating any potential incidents. By decoupling deploy and release, feature management also offers a degree of flexibility that the traditional deployment strategies lacked. For example, you could start with a canary to get real user feedback and transition into a rolling release once the canary is complete to reduce the risks of incidents caused by increasing load.

## LaunchDarkly’s Approach to Deployment

Regardless of what strategy you choose, feature flags and feature management can only improve your ability to incorporate testing in production, experimentation and personalization while helping to minimize the risks of incidents and downtime. LaunchDarkly has always been at the forefront of feature management and continues to expand our capabilities in ways that fit the evolving approaches towards deploy and release.

For example, [contexts](https://docs.launchdarkly.com/guides/flags/intro-contexts) broaden the scope of targeting beyond just users to make it easier to target anything, from locations to machines and devices to organizations. This gives you a deeper and more granular control over targeting that can offer you more flexibility in how and when you release features. Meanwhile our [experimentation](https://docs.launchdarkly.com/home/about-experimentation/?q=experim) capabilities offer you the data and guidance you need to not just safely release feature but to release the _right_ features.

Best of all, this is all done through an intuitive interface that developers, product owners as well as other stakeholders can easily understand and manage. Don't take my word for it though, [give it a try for free](https://launchdarkly.com/start-trial/).