# Managing Entitlements in LaunchDarkly

Feature flags might seem like an odd fit for managing entitlements, but ultimately, from an application standpoint, entitlements are about enabling/disabling and/or rate limiting specific features. This is a capability feature flags excel at when combined with advanced targeting capabilities like those available in LaunchDarkly.

There are some important benefits to managing entitlements via LaunchDarkly feature flags:

1. You do not need to alter or move your existing account management solution. While LaunchDarkly can potentially be used as a source of truth for your accounts, the most flexible solution is to handle account management through an internal or third-party tool specifically designed for that purpose.  As we'll see, just passing the proper targeting information for a tenant will enable LaunchDarkly to determine which features to enable/disable or rate limit.
2. It allows you to manage feature access regardless of where it is implemented. LaunchDarkly provides over 25 SDKs for various languages and platforms. This means that the same flags that are deployed to code running on your Amazon EKS cluster or your AWS Lambda functions can also be deployed to your frontend web or mobile app. 
3. The setup is incredibly flexible. While we will demonstrate one way of setting up your entitlements in LaunchDarkly, this is just one of many ways that this can be set up depending on your specific needs.

## The Basic Account Structure

We're going to use a fairly common set up whereby each tenant is assigned to a plan. These plans increase in terms of their access to features, while inheriting the features of all the plans beneath them. In our example, the plans look like this:

- **No Plan** - These are accounts that may be inactive or have very limited access. For example, an account that is no longer active but still retains access to viewing prior account billing information. These will remain unassigned to any segment within LaunchDarkly.
- **Basic plan** - A basic plan may have access to a baseline set of functionality that any user with an active account can access. The point of this grouping would be to distinguish between active accounts and accounts that have no access to application functionality (i.e. the no plan accounts).
- **Pro Plan, Business Plan, Enterprise Plan** - Each of the remaining account plans has an incrementing amount of access to features, meaning that they get the features of all the plans beneath them as well as additional features or increased access to rate limited features.

One critical thing to keep in mind is that, while broader access to features will be determined by a tenant's plan, you have a lot of flexibility. As we'll see, you could still assign individuals, or groups of individuals access or even a tenant access to a feature not in their plan. For example, if your contract with a particular Business Plan customer gave them access to an enterprise feature, the system would still be capable of handling those exceptions.

### Creating the Feature Flags

Within each plan there are two types of features:

* **A toggled feature** – this feature is disabled (off) for one plan but enabled (on) for another. These will use a standard boolean flag that will indicate whether the feature is on or off for a given plan.
* **A multi-variate feature** – this is a rate-limited feature that may be enabled for some of all account levels, but to varying degrees. For example, you might offer a very limited number of API calls to a basic plan but increase or even remove those limits with each successive plan.

The toggled features will be dependent flags of a flag representing the plan that they are associated with. The benefit of this is that it allows targeting of the parent flag to enable access to all the dependent features without needing to target each individual feature flag. We'll explore how targeting works later in this article.

Here's an example of what this might look like:

```
.
├── Basic Plan
│   ├── concurrent meetings: 1
│   └── transcription minutes: 300
├── Pro Plan
│   ├── speech to text
│   ├── automated summary
│   ├── concurrent meetings: 2
│   └── transcription minutes: 1200
├── Business Plan
│   ├── sync files
│   ├── API access
│   ├── concurrent meetings: 3
│   └── transcription minutes: 6000
└── Enterprise Plan
    ├── live captioning
    ├── single sign on
    ├── concurrent meetings: 3
    └── transcription minutes: 6000
```
To start, you'll want to create the parent flags before you can create the child flags for toggled features.

### Creating a parent flag or a toggled feature flag

Creating a flag for a parent flag or a toggled feature is straightforward:

1. Within the LaunchDarkly console, click the "Create flag" button.
2. Give the flag a name. The flag key will be created for you automatically based upon the name. You can also optionally add a description.
3. Optionally, we can add a tag to make this flag. For example, we might want to tag it as "entitlement" to distinguish this from other flags within our system unrelated to entitlements.
4. If this flag will be used on client-side browser applications or on mobile applications, be sure to check the checkboxes to make it available to those SDKs.
5. Leave the boolean flag variation options with their defaults.
6. Check the "This is a permanent flag" option. This simply prevents the LaunchDarkly console UI from prompting you to remove the flag.
7. Click "Save flag".

![creating a flag](creating-a-flag.png)

If this flag is a child of a parent flag representing a plan (for example, the live captioning flag is a child of the Enterprise Plan flag), there are some additional steps.

8. From the flag detail page, be sure you are on the "Targeting" tab. Under the "Prerequisites" heading, click "Add prerequisites".
9. Choose the prerequisite flag from the "Select a flag" drop down. For example, if we're creating the "API access" flag from the example above, we'd choose the the "Business Plan" flag as the parent.
10. The "Select a variation" option should be set to "true".

In both of these cases, once the flag is saved, turn targeting on and save it. Having targeting turned on will allow it to target this flag to our specific segments representing our tenants, which we'll set up later in this tutorial.

![add a prerequisite](add-prerequisite.png)

### Creating a multi-variate feature flag

Multi-variate flags will work differently. Ultimately, each segment we create will be targeted with a specific variation of the flag. For instance, using the prior example, a tenant on the "Pro Plan" will get a value of 3000 from the "transcription minutes" flag while a "Business Plan" tenant will get 6000. Because of this, these flags do not need to have a prerequisite (i.e. parent) flag set.

Follow steps 1 through 4 for creating a toggled feature flag.

5. Under flag variations, choose whether this is a number, string or JSON flag. For example, the  "transcription minutes" flag would be a number.
6. Enter each of the variations. For example,  the "transcription minutes" flag would have variations of 300, 3000 and 6000.
7. Set both the default variations to the value representing the lowest plan.

Once the flag is saved, turn targeting on. We'll see later in this tutorial how we'll set up segments to target specific variations.

## Segments

1. Choose the segments menu item on the left-hand side of the LaunchDarkly console.
2. Click the "Create segment" button.
3. Give the segment a name - for example, "Enterprise Plan". The key will be generated for you based upon the name. You can also optionally add a description.
4. Choose the "Standard" segment option and then click "Save segment".

We'll repeat this process, creating a segment representing each plan within our entitlements structure.

## Contexts

dig into security features (secure mode https://docs.launchdarkly.com/sdk/features/secure-mode)