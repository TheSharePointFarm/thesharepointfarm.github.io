---
layout: post
title: SharePoint Server 2016 Outgoing Email and Exchange Receive Connectors
tags: [sp2016]
---

There is currently a bug in SharePoint 2016 with regards to Outgoing Email and Exchange Receive Connectors. Prior to SharePoint 2016, generally no changes were required to send mail to a Receive Connector, but in SharePoint Server 2016, Microsoft switched to using the [SmtpClient](https://msdn.microsoft.com/en-us/library/system.net.mail.smtpclient%28v=vs.110%29.aspx) namespace for sending mail, and in doing so set [UseDefaultCredentials](https://msdn.microsoft.com/en-us/library/system.net.mail.smtpclient.usedefaultcredentials(v=vs.110).aspx) to true. This means any Receive Connector which has an authentication mechanism attached to it, SharePoint will attempt authentication and fail.

The solution, for now until this issue is resolved in SharePoint, is to create a new Receive Connector in Exchange and set the Authentication to Externally Secured and the Permission Group to Exchange Servers. In addition, it is recommended to restrict the Receive Connector to the IP addresses of your SharePoint servers to prevent any other service from relaying through that Receive Connector.