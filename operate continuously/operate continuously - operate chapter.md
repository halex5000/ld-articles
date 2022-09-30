operate

What is an incident

preventing incidents by mitigating risk

- roll backs
- kill switches
	- Technical reasons versus business reasons
	- LD case study
- safety valves
- monitoring
	- human versus automated
	- identifying problems
	- Feature flags + monitoring



In the days of shipping software on CDs in boxes, software releases were “done” once the CDs hit the users’ hands. Now, with software delivered through the cloud and hosted remotely, there isn’t an equivalent sense of “done”. Complexity abounds and entropy encroaches. The interplay of your code, user behavior, 3rd party integrations, internal tooling, cloud hosting and SaaS vendors mean that your software can perform unexpectedly at any time. 

There are two sides to dealing with the inevitability of incidents. The first involves what you do before an incident occurs, since they will occur. The second is how you handle an incident after it happens.

Before an incident occurs, you want to be sure that you create processes that limit their impact on both your application and its users as well as on your organization. We'll discuss some strategies that can help you and your team proactively shorten the length and reduce or eliminate the damage incidents may cause.

After an incident has happened, you need strategies for identifying the problem, fixing it, testing the fix and finally releasing the fix. Since incident responses require the involvement of technical and/or other staff that can identify and respond to incidents, we'll discuss topics of people management and culture that can improve your incident response processes as well as tools that can help.

After an incident, it is always valuable to reflect on what happened, how well you and your team responded and ways that you can improve. So we'll look at some best practices for post-indident retrospectives and how to establish and measure metrics that can help you determine the effectiveness of your incident response processes.

## What Is An Incident?

OPERATIONS IS NOT JUST INCIDENTS

An incident is any event that requires immediate, out-of-band remediation. Urgency and customer impact should be the barometers of declaring an incident.

Incidents are not just system outages. Incidents can and should be declared if there is a problem in your system that might lead to an outage, degraded performance or even prevent access to application functionality, even if customers are currently unaffected-- this is an essential shift towards moving to a proactive incident management process that can detect and mitigate incidents before customers are even aware.

### Common incident causes

Incidents typically have one of three causes:
1. Changes to your system
2. Changes to the inputs of your system
3. Non-software issues

#### Changes to your system

Almost any software system is in a fairly constant state of change, from infastructure changes that can include both hardware and software updates to new features being built and deployed by your development team. Any of these items may cause an incident to occur. In fact, according to The Visible Ops Handbook, "80% of unplanned outages are due to ill-planned changes made by administrators ("operations staff") or developers." (https://itpi.org/the-visible-ops-book-series/visible-ops-handbook-review/)

When it comes to the software we build, continuous deployment makes your life easier. If you are deploying multiple times per day, each of those changes will individually be minor in size and importance. If part of your application starts failing, the most recent deploys are the most obvious culprits. Smaller deploys make roll backs or kills switches, which we'll discus later, easier, which means easier incident management. Each individual deploy has less complexity to untangle, and a more narrow scope.

Frequent updates and deploys, even in small batch sizes, can lead to a higher likelihood of small incidents, as any deploy can have unintended consequences. These small incidents can be handled more easily, as they tend to not be as complex and are easy to isolate. We'll discuss some strategies to minimize the risk of these deploys in this chapter.

#### Changes to the inputs of your system

Changes to the inputs of your system also cause incidents. These can include a sudden surge in traffic, new customers or existing customers using your software in ways you didn’t anticipat, and unexpected changes or outages to third-party integrations or APIs.

Just as incidents are inevitable for you, they are for your vendors as well. Sometimes a vendor will have a major enough incident to impact your system in a way you weren’t prepared for, and their incident will have now caused your incident.

Your customers can also cause incidents by using your software in unpredictable ways. You want lots of customers, and you want those voluminous customers to use your software constantly. If this happens, you will have found strong product market fit, congratulations! The down side of thousands or millions of customers using your software all the time is that they will find new ways of using your product that cause things to break. However robust your QA and testing process might be, your customers can and will do things you don’t expect, and this will result in incidents.

#### Non-software issues

The rest of the incidents are caused by non-software issues. Creating software comes with inherent risks and pitfalls, but at least code is orderly and readable. The rest of the world outside of our computers can be quite messy.

Happy employees can lose laptops or entry badges. Unhappy employees can publish unsavory stories that threaten your company’s reputation. Global health pandemics like Covid-19 can quickly disrupt your standard business operations.

These aren’t code-related, but these “real world” issues can cause major technical damage to your organization’s ability to deliver a stable product, and should be accounted for in your incident management process.

## Mitigating risk

Now that we've idenfied the risk, let's talk about how we can deal with it ahead of time. The processes and tools you put in place before the inevitable incident can help to reduce the frequency of incidents, their length and the damage they may cause. These can include:
* Managing roll backs when needed
* Implementing kill switches or safety valves
* Planning migration strategies
* Implementing observability

When the right process for each of these can reduce the time, cost and damage of managing incidents. In this section, we'll explore each and offer advice on how to establish each effectively within your organization.

### Rollbacks

Rolling back to a previously known working version can mitigate the disruption and allow the teams to troubleshoot and then fix the problem. However, rollbacks can sometimes be difficult, particularly if you release in large batches or when database changes are involved. As we discussed, continuous deployment can keep batch sizes small and reduce the complexity of rollback, but, in addition, release strategies like blue greens or canaries and the use of feature flags can also help to decrease the disruption caused by a rollback.

Blue green and canary deploys are designed to help minimize the need for rollbacks by allowing you to test code in production before releasing it to all users, however, when a rollback is needed, they also can ease the pain of reverting back to the prior version.   In case of a blue green deploy, the change is not so much “rolled back” as all the traffic is diverted back to the prior version. Depending on how your canary deploy is implemented, you may need to roll back deployed changes but rollback is simplified by the fact that the new version and prior version coexist, so users who’ve been accessing the new version will need to be redirected back to the original. While both strategies provide easier rollbacks, keep in mind that database changes can potentially complicate either one.

### Kill switches 

A physical kill switch is designed to shut off machinery in the case of an emergency. The concept is basically the same when we’re talking about software. The easiest way to implement a kill switch in software is using feature flags.

A kill switch completely disables a feature or set of features. If something isn’t working, using a kill switch doesn’t fix the underlying problem, but it does disable the code within the application, minimizing its damage. This allows for a near immediate response to an incident once reconized and diagnosed, without the need to perform a rollback.

The flipping of a kill switch can be triggered by a human or automation. Humans can shut off features, customers or integrations that are causing problems whenever the need arises. Automations can do this as well, particularly when connected to application performance monitoring (APM) tools (more on that a bit later).

It's also worth noting that a kill switch does not necessarily need to turn off a feature for everyone. Feature management platforms offer user targeting capabilities that can allow you to turn off a feature for a specific subset of users that are affected by an incident. For example, if your application has a bug that only affects Android users, you may consider turning it the relevant feature off only for those specific users.

When explaining software development and operations to non-technical people, the idea that you can simply turn off something that isn’t working seems pretty obvious. Feature flags make kill switches easy to implement but also make the act of turning off a broken piece of software relatively quick and painless. You do not need to rollback a large part of the application by reverting to the last known working version, which can often mean rolling back adjacent functionality. This makes the decision less impactful, meaning fewer approvals and fewer trade-offs, which lead to fewer delays in resolving the incident.

#### Turn off features for business reasons

Kill switches don’t necessarily need to be limited to technical issues but can also be used if business issues arise. Sometimes the software works perfectly, but other factors such as unexpected demand, unpredictable weather, complexities in the supply chain or any number of other business reasons necessitate temporarily shutting off access to a feature. 

For example, a well known fast food chain  rolled out a new item on their menu that proved immensely popular. Predicting regional demand for physical goods can be tricky, and the item sold out in some places but not others. When the item was no longer available, the company used a kill switch implemented with a targeted feature flag that turned off the menu item in the places where it was no longer available. This reduced customer frustration as hungry customers weren’t tempted by what wasn’t available. Modern feature management systems have extensive user targeting capabilities that can enable turning on and off features based upon configurable properties or identifiers, such as geolocation. 

#### Kill switch case study

Kill switches don’t make the problems disappear, but they do minimize the blast radius and allow problems to be quietly fixed behind the scenes. To see how this works in the real world, I'd like to share an actual experience from our team at LaunchDarkly. LaunchDarkly sells feature management software that allows customers to create and use kill switches, but sometimes we also have to use them ourselves.

One time we released a feature that inadvertently caused Safari users to lose critical access to updating LaunchDarkly feature flags. This was on a Friday. Some organizations consider Friday releases to be a bad idea, because creating weekend incidents can lead to bad outcomes and low morale. Without kill switches, releases can have giant, weekend-destroying consequences. With kill switches, the risk is much lower.

A customer sent us a support ticket indicating they were unable to flip feature flags in LaunchDarkly’s user interface on Safari. The on-call staff confirmed the bug in our development environment and that it affected all Safari users. The next step was to determine which feature was causing the issue.

We reviewed the last few things we released and found a likely culprit. Turning the flag for that feature off in our development environment restored the Safari functionality, so the team felt comfortable that turning the flag off in production would address the issue.  Our support team reached back out to our customer and confirmed that the issue was resolved. Finally, we scheduled a bug fix in the next week's iteration plan to allow us to rerelease the feature.

The mood would have been quite different if reverting back to the previous working version meant reverting days, weeks or months’ worth of changes. This would have increased the scope and impact, meaning more people would have to be involved in making the decision to roll back. This also would have taken longer, widening the impact on customers, meaning more support tickets from users who are understandably frustrated that the issue is preventing them from accomplishing their jobs.

Remediating incidents is challenging under the best of circumstances, particularly with customers and stakeholders demanding answers instantaneously. Feature flagging helps address these challenges by offering multiple advantages to the traditional way of doing things:

1. It gives you the confidence to do things that would otherwise be considered risky, such as releasing on a Friday, because it lowers the impact of failure.
2. It offers much faster troubleshooting because you can very rapidly isolate the impact of any specific change within a group of changes without redeploying, both in your development and production environments.
3. Once the culprit has been discovered, you can leave it in a disabled state and solve the problem in the normal course of business, rather than an emergency deployment or emergency patch. This means bug fixing becomes more routine, less urgent, and saves on costly incidents involving numerous stakeholders.

In this case, kill switching a release was pretty low effort and low stakes. Kill switches give your infrastructure choke points that can be utilized to turn off portions of the software that threatens to impact your users or even other parts of the application.

### Safety valves

As the name implies, a safety valve is designed to relieve pressure in order to avoid a potentially significant problem. They bear a lot of similarities to kill switches in that they:

* Can be implemented in software using feature flags
* Can be triggered manually or via automation, for instance when connected to APM tools
* Can disable or throttle access to a feature to address an actual or potential incident

Key differences between safety valves and kill switches are that:

* A safety valve typically doesn't completely remove the feature but throttles access to it
* Safety valves are usually associated with an external dependency
* Because they are associated with a dependency, safety valves are typically a long-living or even permanent flags, whereas a kill switch is usually removed once a release has been deemed successful.

For example, perhaps your application relies on a non-critical third-party service that has been having issues with performance or reliability, a safety valve can be used to help gracefully degrade the application in order to deal with outage or availability issues. A problematic service or feature can be feature flagged as a safety valve that is triggered when certain conditions arise thereby limiting access or usage of features that depend on this particular dependency in order to avoid a larger outage.

### Handling migrations

Migrations involve moving from one service, software or infrastructure to another and are notoriously risky to release. In many, if not most, cases, a migration is transparent to the users. This makes a progressive rollout one of the most effective ways to handle the release of a migration. A progressive rollout slowly pushes more and more traffic to the new service. While your users (hopefully) won't see the difference, it gives your team time to verify that the migration under increasing load and revert before any potential incident occurs. 

Progressive rollouts are a key use case for feature flags within a migration project. The flag can randomly assign a percentage of users to the new service. After a pre-determined amount of time, if no issues arise, a larger percentage of traffic can be routed to the new service until ultimately it is rolled out to the entire user base. While this may be handled manually, some feature management platforms even allow this process to be automated.

Using feature flags and progressive rollouts for migrations can bring about larger organizational benefits as well. In many cases, teams avoid beneficial migrations simply due to the risk. By decreasing the risk of a migration, they can increase your team's willingness to take on a migration project. This can ultimately lead to more frequent improvements to the services, software or infrastructure that underly your applications, which can mean they perform better or even lower the costs of managing them.
