---
layout: default
title: SharePoint Test Environments
---

This may not work for everyone, but this is the methodolgy I personally use.  I have two test environments, a development environment and a pre-production environment.

Production environment is in proddomain.com, pre-production is in preproddomain.com, and finally development in devdomain.com.  Preproddomain and Devdomain have a one-way trust with proddomain.com (they trust proddomain.com).

Building out the environments is done as accurately to production as possible, of course.  Even the account names are the same between all three enviornments (although this isn't required, and if you're not careful, can cause lockouts depending on what you're doing (looking at you, Axceler ControlPoint!).

SharePoint is installed via a PowerShell script to make Service Application names the same, etc.  Web Apps are also created via script for the same reason.  AAMs are obviously different (e.g. webapp.preproddomain.com instead of webapp.proddomain.com).  SSL is still set up, thanks to a wildcart certificate on the WFE, again using a different FQDN between each enviornment but same hostname.

Patching is done via the same methodology that was performed in production (e.g. started off with SharePoint + Feb 2011 CU; Project added on top of that, re-patched, OWA added after that, SP1, then June CU).

Finally, restore the Content Databases only to the pre-production/test SQL Servers.  Use a Mount-SPContentDatabase with a user that has sysadmin privileges on SQL Server.  This will add all the required permissions to the database and attach it to the specified Web Application.

End users should still be able to access the SharePoint Sites via their production domain credentials.  I personally give my pre-production account Full Control on each Web Application and use that primarily instead of my production account.

The disadvantage of using different Active Directory forests for each environment is the People Picker and Project Server.  Because one-way trusts are enforced, Project Server will not work correctly without some work-arounds.  Namely, add the user to the Project Server SharePoint Site Group first, then to Project Server.  For each Web Application, the peoplepicker-searchadforests property must be implemented.  The other route to take is to create accounts in the pre-production/test domains and add those accounts to Project Server instead.

This method has worked and given us the isolation between environments as we require them (thanks, SoX!).

This speeds up deployments, and prevents any conflicts between the various environments, while "following" the production environment with regards to patch levels.  Once the test environment is fully up and running, production starts to lag behind the test environment as setting changes are made, patches released, and so forth.