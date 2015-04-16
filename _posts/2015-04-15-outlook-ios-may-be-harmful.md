---
layout: post
title: "Outlook for iOS and Android potentially harmful"
author: Joseph Turner
tags: outlook ios threat credentials
category: compromise
---

*This is the first of a series of posts on threats and risks we've found
using [Interlock](http://www.mobilesystem7.com/interlock/).*

A few days ago, we were walking a customer through the results of their
Interlock Test Drive. While examining one of the Bad users, the customer interrupted the
demonstration, saying "That's not right. That user shouldn't be in Oregon." We
took a closer look at the identity in question, and found something
troubling: the user's internal, Exchange email account was being logged into by an
automated system hosted in AWS.

A bit of background. Interlock uses advanced [Identity
Analytics](/blog/post/identity-analytics/) to
evaluate the [risk each identity poses](/blog/post/a-treatise-on-risk/) to the organization based on
their characteristics and activities over time. Identities that pose a
large risk are flagged as Bad and bubbled up to the user interface to be
evaluated (though they can also be [mitigated
automatically](/blog/post/adaptive-access-control/) or
generate out-of-band alerts). The identity we were looking at had been
flagged as bad precisely because it looked like abnormal access: out of
the user's and organization's normal operating locations, from a new IP
assocated with a
new location some distance away, with a new device type.

We ran a WHOIS search on the IP address of the access, and it was an AWS
account.

    NetRange:       XXX.XXX.0.0 - XXX.XXX.255.255
    CIDR:           XXX.XXX.0.0/12
    NetName:        AMAZON
    NetHandle:      XXX-XXX-XXX-XXX-XXX
    Parent:         XXXX (XXX-XXX-XXX-XXX-XXX)
    NetType:        Direct Allocation
    OriginAS:
    Organization:   Amazon Technologies Inc. (AT-88-Z)
    RegDate:        2014-10-23
    Updated:        2014-11-13
    Ref:            http://whois.arin.net/rest/net/XXX-XXX-XXX-XXX-XXX


    OrgName:        Amazon Technologies Inc.
    OrgId:          AT-88-Z
    Address:        410 Terry Ave N.
    City:           Seattle
    StateProv:      WA
    PostalCode:     98109
    Country:        US
    RegDate:        2011-12-08
    Updated:        2014-10-20
    Comment:        All abuse reports MUST include:
    Comment:        * src IP
    Comment:        * dest IP (your IP)
    Comment:        * dest port
    Comment:        * Accurate date/timestamp and timezone of activity
    Comment:        * Intensity/frequency (short log extracts)
    Comment:        * Your contact details (phone and email) Without these
    we will be unable to identify the correct owner of the IP address at
    that point in time.
    Ref:            http://whois.arin.net/rest/org/AT-88-Z

    OrgAbuseHandle: AEA8-ARIN
    OrgAbuseName:   Amazon EC2 Abuse
    OrgAbusePhone:  +1-206-266-4064
    OrgAbuseEmail:  ec2-abuse@amazon.com
    OrgAbuseRef:    http://whois.arin.net/rest/poc/AEA8-ARIN

    OrgNOCHandle: AANO1-ARIN
    OrgNOCName:   Amazon AWS Network Operations
    OrgNOCPhone:  +1-206-266-2187
    OrgNOCEmail:  aes-noc@amazon.com
    OrgNOCRef:    http://whois.arin.net/rest/poc/AANO1-ARIN

    OrgTechHandle: ANO24-ARIN
    OrgTechName:   Amazon EC2 Network Operations
    OrgTechPhone:  +1-206-266-4064
    OrgTechEmail:  aes-noc@amazon.com
    OrgTechRef:    http://whois.arin.net/rest/poc/ANO24-ARIN

Not good. Somehow, a system in AWS had the identity's credentials and
was logging into the organization's Exchange account. Digging in deeper,
we looked at the claimed User Agent string of the activity and found that it was claiming to be
`Outlook-iOS-Android/1.0`. A Google search turned up [this
article](http://blogs.office.com/2015/01/29/deeper-look-outlook-ios-android/),
indicating that the User Agent string is associated with the new (at
least to Microsoft, more on that in a moment) Outlook for iOS and
Android apps (or something masquerading as such). So why is it in AWS?

Rather than building the apps from scratch, Microsoft [purchased Acompli](http://blogs.microsoft.com/blog/2014/12/01/microsoft-acquires-acompli-provider-innovative-mobile-email-apps/),
the company responsible for the app. In acquiring the company, they also
acquired the legacy architecture, which currently resides on AWS (though
surely they are furiously moving it over to their own infrastructure!).
Reading the comments on the article above, we found other enterprise
users who had noticed traffic from these apps and were concerned. This
led us to the [Acompli Privacy
Policy](https://www.acompli.com/privacy-policy/), which includes the
following lines:

    We collect and process your email address and credentials to provide you the Service. 

and

    We collect and process your email messages and associated content to provide you the Service. 

In other words, in order to provide the quality of service they wanted
for Outlook for iOS and Android, they are caching both credentials and email
data on their own servers.

For the casual, personal user of Office365, this is probably no big deal, but
for enterprise customers, particularly ones constrained by compliance
requirements, this may be unacceptable. Enterprise data and credentials
are being **copied and stored on third-party servers.** This is
a violation of the security policies of many (most?) organizations.

According to the comments by Microsoft personnel, these are valid
concerns. Organizations who want to limit their
exposure to this issue can use the [ActiveSync Allow/Block/Quarantine
list](http://blogs.technet.com/b/exchange/archive/2010/11/15/3411539.aspx)
feature of Exchange 2010+ (or see [this
article](http://blogs.technet.com/b/exchange/archive/2008/09/05/3406212.aspx)
for older versions) to limit or block the access of this app. Of course,
this may still result in credentials being cached even if they are
blocked by Exchange, so some user education is probably also in order. 

If you're interested in finding potential threats like this
automatically as they arise in the future, please [contact us](http://www.mobilesystem7.com/test-drive/) to take
a test drive of Interlock.
