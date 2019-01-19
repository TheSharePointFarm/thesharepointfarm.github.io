---
layout: post
title: Random SharePoint Administration Thoughts
tags: [sp2010,sp2013sp2016]
---

Random SharePoint Administration thoughts... In no particular order.

[Disable SSL 3.0]({% post_url 2015-08-05-sharepoint-support-for-disabling-ssl-3-0-and-tls-1-0 %}) in SharePoint 2010 and 2013; SSL 3.0 through TLS 1.1 for SharePoint 2013.

Use SSL everywhere. There is no reason not to. [Use SNI](https://en.wikipedia.org/wiki/Server_Name_Indication#Support), if appropriate. All browsers supported by SharePoint 2016 also support SNI. Certain browsers supported by SharePoint 2010 and 2013 do not support SNI, but you shouldn't be using those anyways.

Use SSL Bridging, never SSL Offloading. SSL Bridging is a "man-in-the-middle" appliance that decrypts the user's session but re-encrypts the session before sending it to the destination server. SSL Offloading should be considered insecure for SharePoint. Services like Office Web Apps 2013, Workflow Manager 1.0, Exchange Task Sync, and SharePoint Apps leverage OAuth2 tokens. OAuth2 tokens are [insecure and can be replayed]({% post_url 2014-09-18-dangers-allowhttp-sharepoint %}) if intercepted.

Use the Windows Firewall. There's no reason to not do so. SharePoint creates (most) the Windows Firewall exceptions.

Think twice about using SAML. There are many user experience issues (namely the People Picker) and Business Intelligence is problematic. If you want to use ADFS for Single Sign On purposes, use ADFS 3.0 (Windows Server 2012 R2) using a non-claims aware relaying party. This allows you to continue using Windows Claims on SharePoint.

Use Kerberos on SharePoint Web Applications. It's simple! `setspn -S HTTP/webAppUrl.company.com DOMAIN\iisAppPoolAccount`. And of course make sure Kerberos is selected in the Authentication Provider settings. Kerberos is faster and more secure than NTLM. *certain cross-forest restrictions may apply

And speaking of Kerberos, always enable Kerberos for SQL Server. Again, simple. `setspn -S MSSQLSvc/sqlServerName.company.com:1433 DOMAIN\sqlServiceAccount` and `setspn -S MSSQLSvc/sqlServerName:1433 DOMAIN\sqlServiceAccount`. Just remember, local connections to SQL (e.g. running SSMS on SQL Server) will continue to use NTLM authentication.

Stop Service Instances and delete Service Applications you're not using. Not sure what you'll use on a new deployment? Only deploy Managed Metadata Service, Search Service, and the User Profile Service Application. Nothing else. Start Service Instances and create Service Applications as business requirements call for it.

Keep [Web Applications to a minimum]({% post_url 2014-08-25-expense-application-pools %}). Two is preferable when MySites are being used (one standard Web Application, the second dedicated to MySites).

Keep Application Pools to a minimum. A single Application Pool for all Web Applications, and a single Application Pool for most Service Applications.

Keep Managed Paths to a minimum, with a maximum recommended of 20. Each request is evaluated against the Managed Paths, so the fewer you have, the better the performance.

Consider following the [Streamlined Topology architecture]({% post_url 2014-08-27-streamlined-topology-performance %}). Sure, it isn't 100% reliable with SharePoint 2010 and 2013 (due to the topology service sometimes routing requests to another server anyways), but in SharePoint 2016, the topology service is "smart" and will route requests locally when the service runs locally.

Not using the User Profile Synchronization Service? Your Farm Admin service account does not need Local Administrator rights! Using the Claims to Windows Token Service? Create a _dedicated_ service account for it. This account _must_ have Local Administrator rights (along with specific User Rights). No other account needs Local Administrator rights.

Always run the Foundation Web Service on all servers. It helps with solution deployment and Security Token Service issues, plus there is no reason to not do so. Have some ISV telling you that you need licenses for all servers running Foundation Web? Just tell them that you only have n number of services filling that role in the farm, they'll often understand and make an exception.

Always test SharePoint security hotfixes. Those hotfixes contain fixes from non-security updates, typically from the previous month. This means security updates may introduce regressions, or just outright break things.

Don't use Microsoft Network Load Balancer. It's a poor solution for your load balancing problems. If free is a big deal, consider something like Apache's mod_ssl or HAProxy. There's plenty of fully free solutions.

Those are just some of my random thoughts about to-do and not-to-do if you're building or maintaining a farm. What would you add to this list? 