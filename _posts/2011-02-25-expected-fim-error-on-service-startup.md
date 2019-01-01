---
layout: post
title: Expected FIM Error on Service Startup
tags: [sp2010]
---

The following error is misleading.  It is expected on FIM startup (currently using the December 2010 CU) and can be ignored.

{% highlight text %}
Log Name: Application
Source: Microsoft.ResourceManagement.ServiceHealthSource
Date: 2/25/2011 7:03:20 AM
Event ID: 22
Task Category: None
Level: Error
Keywords: Classic
User: N/A
Computer: APPLAYER
Description:
The Forefront Identity Manager Service cannot connect to the SQL Database Server.

The SQL Server could not be contacted. The connection failure may be due to a network 
failure, firewall configuration error, or other connection issue. Additionally, the SQL Server connection information could be configured incorrectly.

Verify that the SQL Server is reachable from the Forefront Identity Manager Service 
computer. Ensure that SQL Server is running, that the network connection is active, and that the firewall is configured properly. Last, verify the connection information has been configured properly. This configuration is stored in the Windows Registry.
{% endhighlight %}