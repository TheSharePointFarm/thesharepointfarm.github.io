---
layout: post
title: Access Request Emails Failing to Deliver
tags: [sp2013]
---

Access Request Emails are useful tools for Site Administrators to know when a user needs access to a particular resources. It allows them to receive near-immediate notification of the request as well as approve the request quickly from a link in email. But what do you do when they are failing to deliver to their intended recipient(s)?

Sometimes the access request email doesn't make it to the administrator. Either access request emails aren't enabled at the site level, or the email address isn't correct, or the outgoing email settings are wrong. And sometimes all of the information is valid! Obviously the first location to look is in the SharePoint ULS logs. Using ulsviewer, filter by the category "E-Mail" and have a user send a test access request (or reply to in a pending access request). This should confirm whether or not the email is leaving SharePoint. If SharePoint reports the mail successfully sent, then we need to start snooping through the Exchange logs and perform some message tracking. To do this, we will need to use the Exchange Management Console in order to track the message.

```powershell
Get-MessageTrackingLog -Start "3/29" -End "3/30" | ?{$_.EventId -eq "FAIL"} | fl *

...
Source                  : ROUTING
EventId                 : FAIL
MessageId               : <eb7d49a841df41e39c7f84edf412a7a2@EXCH01.nauplius.local>
Recipients              : {TestingGroup@nauplius.local}
RecipientStatus         : {[{LRT=};{LED=550 5.7.1 RESOLVER.RST.AuthRequired; authentication required};{FQDN=};{IP=}]}
TotalBytes              : 2690
RecipientCount          : 1
MessageSubject          : cuser79 wants to access 'Team'
Sender                  : cuser79@nauplius.local
```

In this example, I know there is only one message with an EventId (status) of FAIL, so you'll want to change up your search parameters appropriately. But here we can see that the RecipientStatus (error) is due to authentication required. When SharePoint sends out an email, it sends it anonymously, to an SMTP relay service or perhaps directly to an Anonymous Receive Connector on Exchange. By default, Distribution Groups in Exchange require the sender of the mail message to authenticate successfully with Exchange. This can help clamp down on external spam and the like. Unfortunately, this isn't compatible with the way SharePoint sends out mail, thus we need to open up this particular group, TestingGroup, to all senders.

```powershell
Set-DistributionGroup -Identity TestingGroup -RequireSenderAuthenticationEnabled $false
```

After this, any anonymous user will be able to send mail to this group, and all members of the group will get the Access Request from SharePoint.