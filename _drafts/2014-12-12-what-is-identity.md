---
layout: post
title: What is identity?
author: Joseph Turner
tags: data identity
category: data
---

How do we prove our identity? What makes us unique, and how can we
display that uniqueness to others? As animals, our physical bodies, our
genetic makeup offers one answer, but it is potentially
falliable: what about identical twins?  If two identical twins went on a
camping trip and only one returned, how could we tell which twin it
was?<sup><a href="#1">1</a></sup>
And beyond identical twins,
biometric identity verification would be awkward to implement uniformly
at scale, and it poses privacy issues.

A second possibility is our knowledge, or more broadly, our possessions.
This is what we use most often in our daily lives. We bring our social
security card in to get a drivers license, then we use that drivers
license to open a bank account. We use the credit card we are issued to
make a purchase. When we call the bank, we
recount our last three purchases to prove we are indeed the person named
on the account. Though much more practical, this too is falliable, as evidenced by a neverending rash
of social engineering attacks and stolen bank information. To return to
our evil twin, perhaps you could quiz her on some details of her
supposed life.
This would only smoke out the evil twin in the case where she hadn't
done her research.

How do we catch this devilish [Mary Kate (or was it Ashley?)](https://www.youtube.com/watch?v=CJEoASUMZbI).
The evil twin returns to "her" life,
with a husband and two kids. After a couple weeks, she returns to work.
She seems different at first, but that's to be expected after such a
traumatic experience. After a while, though it starts to get weird. She
quits her book club that she loved so much. She can't remember her
inside jokes with her husband. The songs she sings her kids at night are
different songs. With every day, her real identity becomes more apparent: she
isn't who she says she is.

And this brings us to our third indicator of our identities: our
actions. In everything we do we display facets of our accumulated
knowledge, experience, and idiosyncracies. Individually, these
activities prove nothing, but combined they define our identity in a way
that is very hard to fool. Thought she might hold up to our initial
scrutiny with enough information, it would be basically impossible for
her to act consistently like her sister all day, every day.

## The two aspects of online identity

Online, without physical form, we are all identical twins. Our
identities


Online identities have two faces: identity as a resource and identity as
action [BETTER NAME]. By passing the challenge, whether it's entering your password,
entering a texted auth code, or copying the numbers from your [RSA
token](http://www.emc.com/security/rsa-securid.htm),
you are granted your *identity resource*. The resource allows the
holder to perform the actions granted to that identity. Historically,
much effort has gone into building the fortifications surrounding the
identity resource. Back in the day, enterprises would keep all resources
behind the firewall, require physical presence (or the secret handshake
of a VPN) to even get the opportunity to try and prove your identity.

The move to the cloud has made limiting access in this way trickier.
Without limited access, much work has gone into improving the challenge
itself. Two-factor authentication and related technologies are modern-day
attempts to make access to the identity resource more selective.
Unfortunately, theiy also make it harder to make legitimate identity
claims. Like the firewall, this seems like a tradeoff many companies are
willing to make. Also like the firewall, it isn't foolproof.


<hr />
<a name="1"></a><strong>1: </strong>It turns out that identical twins have different fingerprints, but have
you ever had your fingerprints taken? I haven't.
