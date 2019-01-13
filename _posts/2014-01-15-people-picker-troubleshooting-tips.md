---
layout: post
title: People Picker Troubleshooting Tips
tags: [sp2007,sp2010,sp2013]
---

This is a fairly common issue I see on forums, so I thought I'd have a pre-emptive post that is easy to reference. There are a handful of People Picker issues, where a particular user, group, or all objects cannot be resolved. Here are some common scenarios with easy resolutions.

**General**

When using Windows authentication, only objects with Security Identifiers (SID) can be searched with the standard permissions-based People Picker (Audience Targeting is a different story). This means the object must be a User account or an Active Directory Security account. Active Directory Distribution Lists will never be visible. If you need to be able to email the group, make sure that you use an Active Directory Mail-Enabled Security Group. It has the same functionality as a Distribution List, but it also has a SID.

Validate that you have the [correct ports opened](http://blogs.technet.com/b/wbaer/archive/2009/01/21/people-picker-port-protocol-requirements.aspx) from your SharePoint server(s) to all resolvable Domain Controllers. You can also leverage [automated tools](https://peoplepicker.codeplex.com/) to help you validate this.

You must be using a Domain Account or Machine Account (Network Service, Local Service, LocalSystem (bad admin)) for your Web Application Application Pool in order to query Active Directory by default. Using Local Accounts for your Web Application Application Pool will not work.

And to state the obvious, the SharePoint server(s) must be domain-joined.

**Domain Trusts**

When dealing with Domain Trusts, it is often that there is a One-Way Trust configured (where the domain SharePoint is in trusts the domain where the user resides). In this case, you need to configure the [SearchActiveDirectoryDomains]({% post_url 2014-01-15-powershell-for-people-picker-properties %}) property of the Web Application. With a One-Way Trust of the opposite way, that is, the domain SharePoint resides in is trusted by the domain the user resides in, nothing will work. Look for alternatives, such as a Two-Way Trust or Active Directory Federation Services.

When a Two-Way [Selective Trust]({% post_url 2013-03-20-selective-authentication-can-kill-the-people-picker-in-a-two-way-trust %}) is in place, you must provide the Application Pool Account with access to the Domain Controllers of the trusted domain in Active Directory. See the previously linked article.

Validate NetBIOS name lookups for the remote domain. Yes, SharePoint still uses NetBIOS. [There are workarounds](http://support.microsoft.com/kb/2874332), however.
Missing All or Some Users and/or Groups

Verify that there are no filters on the People Picker queries for the Web Application. This can be validated with [PowerShell]({% post_url 2014-01-15-powershell-for-people-picker-properties %}).

**Troubleshooting**

Quick and easy ways to troubleshoot are if you do not see the troublesome behavior on another Web Application, it is going to be the People Picker Settings or Application Pool account. If you see the issue farm-wide (that is, Central Administration and all Web Applications), it is likely a firewall or name resolution issue.

In addition, often times there will be [ULS entries](http://www.wservernews.com/archives/2013-10-28.htm#EC) about why a call to a particular domain failed. These can be invaluable for resolving People Picker-related issues. Typically it is going to be the stack trace that reveals what is going on, so hopefully you're developer-enough to work through it.

The last invaluable tool to help is a network trace, either [Network Monitor](http://www.microsoft.com/en-us/download/details.aspx?id=4865) or [Wireshark](http://www.wireshark.org/). This is a topic for another blog post, but both of these tools can tell you what is happening under the covers with responses to and from the Domain Controllers.

Feel free to ask here on or on the [MSDN/TechNet SharePoint forums](http://social.technet.microsoft.com/Forums/sharepoint/en-US/home) if you do run into anything you're unable to resolve!