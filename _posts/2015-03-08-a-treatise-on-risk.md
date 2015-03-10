---
layout: post
title: "A (Brief) Treatise on Risk"
author: Joseph Turner
tags: risk identity analytics intrinsic risk unsubstantiated risk
category: product
---

My first week here at [Mobile System 7](https://www.mobilesystem7.com/) was a
crazy one; we spent hours with dry-erase markers in our hands, going over
drawings of architecture, conceptual drawings of data flows and event streams,
a bit of equations, and lots of (probably crucial) stuff that never made it
into my notebook. I knew that our goal was to help organizations identify and
block Bad Guys, but I probably came out of that week thinking that the product
was more about whiteboards than anything else.

Over the months my conceptual model of Interlock has distilled into a better
approximation of its true essence: risk, as it applies to identities, both
[noticing risk](/blog/post/adaptive-access-control/) and [responding to
risk](/blog/post/identity-analytics/). Risk is a challenging problem, seemingly
impossible at first glance. Calculated risk is a measure of not only the
likelihood that a given set of features is going to have a downside, but also a
measure of the severity of that downside. It seems like there are too many
degrees of freedom, that this set of equations has too many unknowns. How can
we even approach this problem, much less do it reliably?

Indeed, it is a challenge to effectively estimate risk, and in a lot of ways it
is both a people problem and a technology problem, but it is in no way
impossible. In order to better understand the overall picture of risk, it is
informative to first separate
[Interlock](http://www.mobilesystem7.com/interlock/)'s view of risk into two
separate concepts: *unsubstantiated risk* and *intrinsic risk*.

## Unsubstantiated Risk

Unsubstantiated risk is the risk posed *by activities* with
unsubstantiated parameters, such as activities with a new device or in a new
location. The important aspect of the unsubstantiated parameters is that they
represent a departure from the user's (or population's) normal operating
parameters. Each activity that has unsubstantiated parameters potentially poses
a threat because it is associated with a nonzero probability that the person
taking the action *is not the person they claim they are*. Think about it: if you
saw someone log into their email from their house, using the same MacBook
they've used every day for a year, at time of day that they log in every day,
it is almost certainly that person; if instead they suddenly logged in from
Sudan using a new Windows PC at a time when they are normally asleep, the
activities performed with those parameters are much more likely to be
malicious. These Sudanese activities pose a large unsubstantiated risk.

It's worth saying that the typical approach to unsubstantiated risk is
to build more structure and trust around authentication, to try and
ensure that the person behind each activity **is** the person they claim
to be. Unfortunately, we have seen time and again that dedicated
attackers can and will gain access to desired resources despite expensive and
complicated authentication measures such as MFA.  As a result, conflating
authentication and security creates a common blind spot for
organizations, one that we talk about a lot: the identity gap. The
identity gap results when authentication is the last line of defense,
leaving each further action subject to no analysis.  Monitoring
unsubstantiated risk for each access goes a long way toward closing this
security gap. Bad guys can no longer act with impunity, as every [action
they take may reveal them](/blog/post/evil-twin/).

## Intrinsic Risk

Intrinsic risk is the risk posed *by the identity* as a result of
its characteristics, things like the total number of devices it uses or the
number of groups it is in, as well as labelable characteristics like that the
user is a frequent traveller. Intrinsic risk adds an element of risk to every
activity associated with the given identity. Each identity with intrinsically
risky characteristics potentially poses a threat because *they have the
capability to be very destructive to the organization*. As a result, every
action they take must be subject to higher scrutiny. Intrinsic risk helps
prevent a number of potential issues, but it's particularly useful in
identifying potential insider threats.

Because it adds risk to activities, intrinsic risk also magnifies any
unsubstantiated risk the identity may be associated with. Because intrinsic
risk implies that the identity has the power to be very destructive, the
implications of a compromised account with high intrinsic risk are dire. For
this reason, intrinsically risky identities with activities exhibiting
unsubstantiated risk represent the largest threat to an organization.

## Trusting Risk

The ultimate goal is not to have the most accurate estimate of
risk for each identity, but to have the most accurate **actionable** estimate of
risk. Each estimate can therefore be associated with a *measure of reliability*
and a *measure of actionability* based on all of the information we have
available to us. Each type of risk has its own methods for computing
confidence, and has different ways of responding to the addition of new
information.

By its very nature, intrinsic risk has a high degree of reliability, but by
itself represents low actionability. Options for remediating intrinsic risk
include destructive actions such as revoking permissions of the identity
permanently, which lowers the intrinsic risk directly, or forcing the
identities to constantly validate their actions. As most organizations have a
spectrum of intrinsic risk across their identities, neither option is
acceptable in general.

In contrast, unsubstantiated risk typically has a low degree of reliability at
first blush, but a high degree of actionability. This is because
unsubstantiated risk can be remediated by allowing the user to substantiate the
parameters of their action. Substantiating actions in this way can happen
automatically or manually. If a user logs into a service from their new phone,
it is an unsubstantiated risk; if they continue to use the phone within
otherwise normal parameters (e.g. logging in from their normal operating
locations at their normal times of day using normal access patterns) that phone
grows to be trusted automatically. Alternatively, assuming the presence of a
trusted communication channel the user can be asked to verify the parameters
directly - did you just use a new Android phone in Brazil?

## Responding to Risk

Ultimately, how we address a given risk boils down to a
combination of the above factors, the degree of each type of risk, the
distribution of each type of risk in the population, and the policies specified
by the user. [Adaptive access control](/blog/post/adaptive-access-control/)
aims to respond to risk in the way that is both least intrusive and most
effective. 

### Unsubstantiated risk

Addressing unsubstantiated risk by asking the user to verify the
parameters of the activities directly offers a one-time process for responding
to risk. If the parameters are indeed correct, the unsubstantiated risk is
attenuated; if not, it is a direct confirmation of a security failure. We are
currently developing a direct solution like this for risk remediation in
Interlock. It promises a tremendous improvement to the focus and accuracy of
our risk estimates with little or no expenditure of scarce security resources
on the part of the organization.

### Intrinsic risk

The correct response to intrinsic risk is usually taking no action at all but
rather subjecting the identity to closer scrutiny. A direct approach to
intrinsic risk is inappropriate. For a potential insider threat the user
herself represents the threat and would be expected to respond negatively; for
a sensitive user, substantiated activities are part of normal operating
procedure. 

In many ways, the biggest challenge is the final factor in responding to risk:
user policies. What tools can we provide the user with to help them understand
the identities in their organization, the scope of their risk, and the best way
to address that risk? How can we leverage the functional differences between
mitigation strategies to effectively and unobtrusively respond to risk? What
granularities of mitigation make sense across organizations and within specific
organizations? These are all questions we are continually striving to answer in
Interlock. Ultimately, organizational risk *mitigation* is a human problem more
than a technology problem. The best we can do is continue to work with
organizations who are struggling with this burden now to find the best solution
for everyone.

If you're an organization interested in finding the best solutions to
addressing risk in your identities [contact us about a free Test Drive of
Interlock](http://www.mobilesystem7.com/test-drive/). If you're a developer (or data scientist, or UX designer!) who
wants to help navigate risk across and among populations of thousands of
identities with hundreds of millions of activities [send me an email](mailto:joseph@mobilesystem7.com) with
who you are what you can add to our team.
