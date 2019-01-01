---
layout: post
title: 
tags: [sp2010]
---

If you attempt to generate a New Key for the Secure Store Service, you may receive the following error: An error occurred during the "Generate Key" process. Please try again or contact your administrator. On the Event Log of the SharePoint server, this error may appear:

```text
Log Name: Application 
Source: Microsoft-SharePoint Products-Secure Store Service 
Date: 8/6/2010 10:29:24 AM 
Event ID: 7557 
Task Category: Secure Store 
Level: Error 
Keywords: 
User: DOMAINUSER 
Computer: SHAREPOINT

Description: The Secure Store Service application Secure Store Service is not 
accessible. The full exception text is: User does not have permission to perform the operation
```text

To resolve this, add the user that is attempting to generate the key to the SharePoint Farm Administrators group.