---
layout: post
title: .NET 3.5 SP1 and SharePoint Search Errors
tags: [sp2007]
---

There are a variety of reasons SharePoint Search fails, but this one is particularly exciting.  Here is the error:

```text
Log Name: Security
Source: Microsoft-Windows-Security-Auditing
Date: 1/27/2009 8:30:04 PM
Event ID: 4625
Task Category: Logon
Level: Information
Keywords: Audit Failure
User: N/A
Computer: SharePointServer
Description:
An account failed to log on.

Subject:
Security ID: NULL SID
Account Name: -
Account Domain: -
Logon ID: 0x0

Logon Type: 3

Account For Which Logon Failed:
Security ID: NULL SID
Account Name: ContentAccount
Account Domain: DOMAIN

Failure Information:
Failure Reason: An Error occured during Logon.
Status: 0xc000006d
Sub Status: 0x0

Process Information:
Caller Process ID: 0x0
Caller Process Name: -

Network Information:
Workstation Name: SharePointServer
Source Network Address: 192.168.0.1
Source Port: 59818

Detailed Authentication Information:
Logon Process:
Authentication Package: NTLM
Transited Services: -
Package Name (NTLM only): -
Key Length: 0

This event is generated when a logon request
fails. It is generated on the computer where
access was attempted.

The Subject fields indicate the account on
the local system which requested the logon.
This is most commonly a service such as the
Server service, or a local process such as
Winlogon.exe or Services.exe.

The Logon Type field indicates the kind of
logon that was requested. The most common
types are 2 (interactive) and 3 (network).

The Process Information fields indicate
which account and process on the system
requested the logon.

The Network Information fields indicate
where a remote logon request originated.
Workstation name is not always available
and may be left blank in some cases.

The authentication information fields
provide detailed information about this
specific logon request.
- Transited services indicate
which intermediate services have
participated in this logon request.
- Package name indicates which
sub-protocol was used among the NTLM protocols.
- Key length indicates the length
of the generated session key. This will
be 0 if no session key was requested.
```

This error was accompanied with the following error in the Application Event Log:

```text
Log Name: Application
Source: Windows SharePoint Services 3 Search
Date: 1/27/2009 8:35:08 PM
Event ID: 2436
Task Category: Gatherer
Level: Warning
Keywords: Classic
User: N/A
Computer: SharePointServer
Description:
The start address <sts3://sharepointsite/contentdbid={625d9a50-2718-4f0e-ad65-bb1c9434f5ec}>
cannot be crawled.

Context: Application 'Search index file on the search server',
Catalog 'Search'

Details: Access is denied. Verify that either the
Default Content Access Account has access to this repository, or add
a crawl rule to crawl this repository. If the repository being crawled is a
SharePoint repository, verify that the account you are using has
"Full Read" permissions on the SharePoint Web Application
being crawled. (0x80041205)
```

Normally, you would make sure the Content Access Account had Full Read, your Alternate Address Mappings were correct, you weren’t pointing to a site with Anonymous Access or SSL enabled, etc.

This particular error is introduced by .NET 3.5 SP1 (this server is running Server 2008).  To resolve it, open `regedit` and navigate to `HKLM\SYSTEM\CurrentControlSet\Control\Lsa`.  Create a new `DWORD` value named `DisableLoopbackCheck`.  Modify the new value and enter a data value of (hex or Decimal) 1.  Restart the Windows SharePoint Services Search service, then run:

```text
stsadm –o spsearch –action fullcrawlstart
```

If you open the Security Event Log, you should no longer see the above error.