---
layout: post
title: "Identity Analytics in Interlock"
author: Joseph Turner
tags: identity analytics machine learning theory
category: data
---

Here at [Mobile System 7](http://www.mobilesystem7.com/), we like to say that we create products that protect your data like
your credit card protects your money. While that phrase evokes the right
ideas - things like constant vigilance, examining multiple factors in determining risk, and 
generally preventing others from pretending to be you - it doesn't
really offer a peek under the hood. In this post, I'm going to talk
about the theoretical basis for the how our [core product, Interlock](http://www.mobilesystem7.com/interlock/), protects your data.

Ultimately, our approach boils down to two related concepts, Identity Analytics and
Adaptive Access Control, and their interplay. Identity Analytics take as input a stream of data
about how a given identity or group of identities interact with the
services they use and output a level of
risk associated for each of those identities. Adaptive Access Control takes that mapping
and changes how the identities can interact with their services, for
example by blocking a high-risk identity's access to sensitive resources.

<img src='/static/img/overview-ia-aac.png'>

Though both parts are integral to how our products work, in this post I'm going to 
talk specifically about Identity Analytics (IA) from a theoretical perspective. First,
I'll break down the IA function pictured above into its
constituent parts. Then, I'll discuss these parts in detail
individually. For the sake of simplicity, I'm going to assume we're
protecting a single identity on a single service. After we get the basics out of the way,
I'll discuss how these ideas can be enhanced when working with a
population of identities across multiple services.

### The basics

Conceptually, the IA function is made up of two parts.

<img src='/static/img/overview-ia.png'>

The first part digests the activity stream and extracts a set of
relevant features. For example, the features might include things like:

  * The identity is in a blacklisted location
  * The identity is coming from a suspicious IP address
  * The identity is attempting to access resources they shouldn't
  * The identity is a member of a privileged group
  * The identity used service X outside of their normal operating time

Why do we need to perform [feature
extraction](https://en.wikipedia.org/wiki/Feature_extraction) at all?
Because although the activity stream arrives as a sequence of discrete events, the
real input to our system at every moment is the **complete activity
stream**, from the beginning of time. This allows us to
understand aggregate usage and behavior. Without examining the full
activity history, we would be forced to evaluate risk based on each
event individually. Using an example feature from earlier, what does
**normal operating time** mean in the context of a single event? In
order for Interlock to be able to use important features like this, it
needs historical data as well. 

By examining the full activity stream, we get a lot more information for
use in our risk level calculation, but it comes at the cost of
processing a *huge amount* of data, much of which is redundant. By performing
feature extraction, we reduce the dimensionality of the data. This
eliminates or aggregates redundant data while highlighting the
information needed by the second part of the IA function: the classifier.

#### A little wrinkle...

Before we move on, I'd like to point out an interesting detail of the
feature observations. Because they are modified when activities arrive,
they technically lives in the [time
domain](http://en.wikipedia.org/wiki/Time_domain); this just means that the values
changes over time. In fact, when a feature is
observed, we model the observation as a function of time as well. In
other words, if an incoming activity causes a feature to activate, the
feature's value will be at a maximum at the time of that activity and
may change as time moves forward. The actual way the
value changes depends on the feature that has been extracted. Some are
fully binary, so when the feature is observed it stays at its highest
value until something mitigates it, like this:

<img src='/static/img/binary-feature.png'>

An example would be membership in a
sensitive group. That feature is full-valued for the entire span of time the identity is associated with the group.

Other features are modeled as "pulses". When the feature is observed,
the value is highest and it decays over time, like this:

<img src='/static/img/pulse-feature.png'>

An example would be when a user attempts to access a resource for which
they don't have permission. Though that feature is relevant to an
identity's risk level today, it is
much less relevant in a week, and even less so in a month. By decaying
the value of the features over time, we ensure that the features
contribute to risk when they are most relevant.

### The classifier

The
[classifier](https://en.wikipedia.org/wiki/Statistical_classification) is a function from the feature vector to a risk level.
Based on customer feedback we
discovered that three levels of risk are enough to accomplish the goals
of the system.

  * `Good` - From the perspective of the IA, this identity is not
    interesting
  * `Suspect` - This identity has performed actions that are abnormal
    and should be monitored more closely
  * `Bad` - This identity is very likely up to no good (e.g., the
    account has been compromised, represents an insider threat, etc.)

The classifier function, then, takes as input a vector of feature values and
outputs one of the discrete classes above.

<img src='/static/img/classifier.png'>

As we just discussed, the features themselves are a function of time, so 
the classifier function also lives in the time domain.
For practical reasons though, the classifier is only invoked
at certain points in time in response to significant changes in the values of the feature vector.
Whenever the risk level is calculated by the classifier for a given point in
time, all feature functions are evaluated at that moment. These slices make up the
actual feature vector consumed by the classifier.

<img src='/static/img/classifier-graphic.png'>

In the figure above, I have annotated the discrete points at which the classifier is
evaluated. The classifier is evaluated when a feature increases in
value, and also when a feature value falls below a threshold. The values
passed to the classifier correspond to the value of each feature at the
point in time when it evaluation is triggered, corresponding to the
vertical lines above. Of course, not every run of the classifier results in a new risk level. 
Practically speaking, there are many more evaluation points than pictured above,
corresponding both to changes in feature value and changes in
metadata about the identity. In general, we want to run the
classifier any time that there might be a change in risk level.

So what is the classifier itself? How does it translate a feature vector
into one of a discrete set of classes? A trivial but still useful approach is a simple test for features:
if **Feature X** is active, return `Bad`. This is an approach taken by many
services today. For example, if you log into your Gmail account while
overseas, you get a notification and possibly a request for two-factor
authentication. This approach can fall short, though, because it makes
it difficult to evaluate features that don't directly contribute to risk
level. 

A more robust classifier examines features not in a
vacuum but in the context of the entire feature set. With this approach,
several features that in isolation have no impact on risk level can combine to affect
risk in a meaningful way. This is the approach we take. Though we are
constantly refining our classifier, as of today it combines an expert-directed decision forest 
with some hand-tuned heuristics. It incorporates feedback both from the
user and from changes in the population of identities to refine the
decisions it makes over time. This system gives us the flexibility to
adjust and modify the feature set and adapt to the requirements of different environments
while remaining confident in the output of the classifier.

### Populations and services

As mentioned earlier, there are a number of practical details that have
been simplified for the discussion above. First, what about populations
of identities? Especially in the enterprise environment, there are
aspects of the group of identities that are relevant to the risk level
for a given identity. A few examples:

  * Accessing resources with more different devices than is normal for
    the organization
  * Operating outside of the group's normal operating location
  * Being in an inappropriately large number of groups

For each organization, the baselines - the normal number of devices, the
organizational operating locations, the appropriate number of groups -
are different. By looking at a group of identities rather that
identities in isolation, we can get tons of useful population statistics
against which we can compare the individual identities. Of course, this
comes at a cost. Instead of processing *merely* the entire activity stream for an
identity, we now must perform feature extraction on the full activity
history for the entire organization.

Moving from a single service to a group of services offers a different
benefit. By examining the actions an identity takes across different
services, we can extract features to build models of typical access
patterns. These models can then identify anomalous behavior that may
represent a threat to that identity or to the organization to which it
belongs. This too comes at a cost. Each additional service increases the size of
the data stream significantly, and there is overhead associated with
extracting and maintaining the intra-service models.

### Conclusions

In this article, I've attempted to outline the theoretical
underpinnings of Identity Analytics as they stand in the current version
of our product, Interlock. While the basic ideas are pretty easy to
explain, the practical issues with feature extraction and classification
are well beyond the scope of this article. Both problems are made more
challenging by the real-time or near-real-time requirements 
dictated by the Adaptive Access Control aspect of Interlock. In future
posts, I'll cover some of these practical issues and how we're addressing them.

Looking forward, our roadmap includes both aggressive improvements to
both our theory and our practice. Improvements in the former will yield
better, more comprehensive pictures of risk for individuals and
organizations, while improvements in the latter will help us reach
bigger scale: larger populations of identity, across even more services.

If you're interested in finding out what our Identity Analytics can tell
you about your organization, please [contact us
today](mailto:sales@mobilesystem7.com) to arrange for a test-drive of
your data in Interlock.
