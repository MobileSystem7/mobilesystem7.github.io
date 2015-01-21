---
layout: post
title: "Adaptive Access Control in Interlock"
author: Joseph Turner
tags: adaptive access control 
category: data
---

In an earlier post I [talked about Identity
Analytics](/blog/post/identity-analytics/), outlining what
they are, why they are a hard problem, and our approach
in [Interlock](http://www.mobilesystem7.com/interlock/).
And while Identity Analytics are useful on their own
- imagine an alert system that offered security officers a low false
  positive mechanism for investigating potentially compromised
identities - their real value emerges when coupled with the other piece
of Interlock:  Adaptive Access Control. Combined, these subsystems
provide a powerful framework for not only understanding risk but reacting to it
in real time. This framework lowers the cost of administration,
reduces user frustration, and most importantly automatically addresses risk as soon as it arises.

So what is Adaptive Access Control (AAC)? In this post, I'll explain
what AAC is in general and why it provides benefits that organization
need. Afterwards, I'll go over the Interlock approach to AAC in increasing
detail, from general concept, down to policies and exceptions, and
finally mitigation steps against individual services.

## What is Adaptive Access Control?

Adaptive Access Control is a way for organizations to differentially control access to
resources and services in a way that incorporates a current view of
risk. More simply put, it changes the way a given user can access
resources when the perceived level of risk for that user changes. This
context-sensitive approach to security is both flexible and powerful. It
lowers the cost of administration by reducing IT support load and
manual security review. It reduces user frustration by lowering the
number of hoops a user must jump through to access resources in the
average case. In the case where there is identified risk, though, it can
provide even greater hurdles for an attacker to access sensitive
services and data. It can also leverage existing mitigation techniques.

As an example, consider a system where users must use [two-factor
authentication](http://en.wikipedia.org/wiki/Two_factor_authentication) each time they log in.
This is costly both in user time
and frustration and in IT support for the system. It also fails to catch
every attacker, because if the attacker can control the two-factor
device they can still access the system. In contrast, with AAC users
with a normal level of risk authenticate normally. Users that exhibit higher than an
acceptable level of risk may be directed through two-factor auth. Users
with a yet higher risk can be denied access to the system
altogether, blocking an attacker even if they had compromised
the two-factor device. This type of AAC is specifically called *adaptive
authentication*, as it alters the way users authenticate based on risk.
As we will see, there are other, finer-grained forms of AAC as well.

## AAC in Interlock

At the risk of stating the obvious, for AAC to work effectively it needs a reliable 
measure of risk for individual identities. This is why the Interlock 
Identity Analytics engine feeds the Adaptive Access Control component.
As a quick refresher, let's review the functional
diagram from the last post. Because there is a bit more input that is
fundamentally relevant, it has been updated to show more detail.

<img src='/static/img/interlock-aac.png'>

The AAC component is driven primarily by three
inputs: the output of the Identity Analytics, which is simply the risk level of the identities 
under management; the available mitigation mechanisms exposed by the
services under management; and the policies specified by the user.
In most cases the latter two inputs change infrequently if at all. If we take
them as fixed, Adaptive Access Control is simply performing actions or
side effects on the services under management in response to risk. What
sort of side effects?

  * Notifying someone
  * Changing user permissions
  * Triggering two-factor authentication
  * Deprovisioning
  * Service-specific measures, such as limiting permissions for specific
    resources

How does Interlock know which action to take for a given risk level change?
That's where policies come in.

## Policies

Policies are the way the user specifies which mitigation strategies
apply to which identities. At the most basic, they define a mitigation
strategy for a given risk level per service. For instance, the policies
for [Okta](http://okta.com) might be:

  * `Bad` users &rarr; Move to group **Bad** in Okta 
  * `Suspect` users &rarr; Move to group **Suspect** in Okta
  * `Good` users &rarr; Do nothing

For each such policy, the defined strategy is applied for the given
service when an identity moves into the given risk level. Each mitigation strategy also includes
an undo mechanism that is applied when the identity moves *out* of
that risk level.

For example, with the policies above when an identity transitions from `Good` to `Bad`,
they will be moved into the group **Bad**. When our `Bad` identity transitions down to `Suspect`, they
will be removed from group **Bad** and added to group **Suspect**. Finally,
when it transitions back to `Good`, they will be removed from group
**Suspect**. Of course, not all mitigation strategies are completely
reversible. For example, reprovisioning a user in Okta requires action
on the part of the user. The user is free to choose fully reversible
mitigation strategies if the desire, but in general this is why the
emphasis is on *side effects* instead of state.

Though the examples above focus on a single mitigation strategy,
policies may specify a group of mitigation strategies. This simply means
that multiple actions are taken when the appropriate transition occurs.
In fact, you can think of the per-service policies as
combined into a single set of mitigation strategies scoped to the
service. Taking multiple mitigation strategies is particularly relevant in the
context of policy *exceptions*.

### Policy exceptions

The simple policies described above assume that the sensitivity and cost associated with
mitigation is the same for all users. In most organizations, this is
not the case. For example, some users have access to more important
data than others. These users have higher *sensitivity* and therefore
may need to be mitigated more aggressively when exhibiting risky
behavior. On the other hand, executives at a company may require access to important
information at all times. They have a higher *cost* of mitigation that
the average user, because mitigating them aggressively will result in
unacceptable loss of access. These users may therefore need to be
mitigated *less* aggressively than normal users.

To specify these types of exceptions, Interlock has a mechanism for
defining policy exceptions for each service. These policies can be
generalized as tuples of `(activity filter, risk level, mitigation
strategy)`. Activity filters are restrictions on the activity to which
the mitigation strategy should apply. The most general activity filters 
refer to identities, such as identities within a certain group or a specific identity.
In the context of this abstraction, a set of policies might look like
this:

  * (Users in group **CxO**, `Bad`, Notify security team)
  * (Users in group **Admin**, `Bad`, Deprovision)
  * (All users, `Bad`, Move to group **Bad**)
  * (All users, `Suspect`, Move to group **Suspect**)
  * (All users, `Good`, Do nothing)

The bottom three tuples of course represent the policies from the
previous section. So how do these get resolved? What happens if the CEO
is also an admin?

### Policy resolution

The only way that policy resolution makes sense with arbitrary
mitigation strategies is to apply them in a specified order. Along
with the tools for creating policies, Interlock also provides a way to
order the policies by preference of application. Policies are then evaluated in the order given,
with evaluation stopping at the first applicable policy.

For the set of policies given above,
this would mean that when a CEO who was also an admin transitioned to `Bad`
Interlock's AAC would only notify the security team and not deprovision
the CEO's account. The desirability of this sort of exception should be
apparent.

### Policy compilation

The underlying mechanism of policy resolution in Interlock is a
series of compilation steps that results in a set of rules reflecting both the
user-defined policies as discussed above, as well as the
current risk level and state of the overall system. These rules are tuples of
the form `(user id, service, mitigation strategy)`. This architecture
is a result of the real-time nature of Interlock, in which we want to
push actions as close to the service as possible (more on that in a bit), and the fact that
the mitigation strategies in most cases represent side effects and not
state as discussed above.

<img src='/static/img/aac-block.png'>

The process of rule compilation is performed whenever there is a change in policies, risk level,
or state, where state refers to any characteristic of 
activities that can be used as a filter in the policies. An example of a
triggering state change would be a user being added to the group **Admin**. As a
result of that change, the policies above would need to be recompiled so
that the rules would reflect the correct mitigation for that user.

## Mitigation

As discussed, the output of the compilation phase is a set of rules that
specify actions for each identity. The final phase of AAC is to
perform the actions specified in the rules. Frequently, the way the actions are taken may
differ from service to service, and the mechanism also may differ by
action taken. For example, to move a user into a group in Okta, an API
request is emitted. If instead we want to deny access to a specific SharePoint resource,
the set of rules is published to the SharePoint agent which can then
deny individual request for the resources. 

In general, there are a set of *adapters* for each service that
understand the requirements for mitigation against those services. These
adapters take the required action to perform the mitigations requested
in the rule set.

## Conclusion

Adaptive Access Control offers a lot of flexibility and power to
organizations looking to protect sensitive services and resources.
Though the basic concept is simple, a surprising amount of sophistication is required
to ensure that the behavior of the system can adapt to the
unique needs of each organization. I've tried to capture the main ideas
and how we address them
in this article, but there are a number of engineering challenges that
arise in areas like efficient policy compilation and specific service integrations that are
beyond what could be covered here. 

One ongoing question for us is how to build interfaces that allow users to
intuitively and efficiently define their policies and ordering,
especially as the number of potential filters, services, and mitigation
options grows. We are actively working on a number of designs to improve
our current implementation in this area. Development in this area, along
with adding new services and mitigation options, will continue to
improve our approach to Adaptive Access Control in the future.

Have questions or comments? Hit me up [on
Twitter](https://twitter.com/josephturnip). Interested in changing the
face of security with us? [Send me your
resume.](mailto:turner@mobilesystem7.com)
