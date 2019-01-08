---
layout: post
title: SharePoint 2013 April 2013 Cumulative Update Fails
tags: [sp2013]
---

When running the Config Wizard for the SharePoint 2013 upgrade with My Sites deployed, you may run across an error similar to this which causes the upgrade to fail:

```text
05/03/2013 22:07:16.26	OWSTIMER (0x1DE4)	0x0854	SharePoint Foundation Upgrade	MySitePersonalSiteUpgrader	0	ERROR	Reset Personal Doc lib permission. Exception: System.ArgumentException: List 'Personal Documents' does not exist at site with URL 'http://mysites2013/personal/testuser2_fabrikam_local'. ---> Microsoft.SharePoint.Client.ResourceNotFoundException: List 'Personal Documents' does not exist at site with URL 'http://mysites2013/personal/testuser2_fabrikam_local'.     --- End of inner exception stack trace ---     at Microsoft.SharePoint.SPListCollection.GetListByName(String strListName, Boolean bThrowException)     at Microsoft.SharePoint.Portal.UserProfiles.MySitePersonalSiteUpgrader.DoMySite_PersonalDocLicPermissionReset(SPSite site)	c401189c-5456-c0fa-03a6-f5fa06142414
05/03/2013 22:07:16.26	OWSTIMER (0x1DE4)	0x0854	SharePoint Foundation Upgrade	SPFeatureDefinition	aj2bj	INFO	SPSite Url=http://mysites2013/personal/testuser2_fabrikam_local	c401189c-5456-c0fa-03a6-f5fa06142414
05/03/2013 22:07:16.26	OWSTIMER (0x1DE4)	0x0854	SharePoint Foundation Upgrade	SPFeatureDefinition	aj2bj	ERROR	Feature upgrade action 'CustomUpgradeAction.MySite_PersonalDoclibPermissionReset' threw an exception upgrading Feature 'MySitePersonalSite' (Id: 15/'f661430e-c155-438e-a7c6-c68648f1b119') in Site 'http://mysites2013/personal/testuser2_fabrikam_local': List 'Personal Documents' does not exist at site with URL 'http://mysites2013/personal/testuser2_fabrikam_local'.	c401189c-5456-c0fa-03a6-f5fa06142414
05/03/2013 22:07:16.26	OWSTIMER (0x1DE4)	0x0854	SharePoint Foundation Upgrade	SPSiteWssSequence2	ajy6m	INFO	SPSite Url=http://mysites2013/personal/testuser2_fabrikam_local	c401189c-5456-c0fa-03a6-f5fa06142414
05/03/2013 22:07:16.26	OWSTIMER (0x1DE4)	0x0854	SharePoint Foundation Upgrade	SPSiteWssSequence2	ajy6m	ERROR	Feature upgrade incomplete for Feature 'MySitePersonalSite' (Id: 15/'f661430e-c155-438e-a7c6-c68648f1b119') in Site 'http://mysites2013/personal/testuser2_fabrikam_local'. Exception: List 'Personal Documents' does not exist at site with URL 'http://mysites2013/personal/testuser2_fabrikam_local'.  (Inner Exception: List 'Personal Documents' does not exist at site with URL 'http://mysites2013/personal/testuser2_fabrikam_local'.)	c401189c-5456-c0fa-03a6-f5fa06142414
05/03/2013 22:08:48.49	OWSTIMER (0x1DE4)	0x0854	SharePoint Foundation Upgrade	SPUpgradeSession	aj0ur	INFO	No context object	c401189c-5456-c0fa-03a6-f5fa06142414
05/03/2013 22:08:48.49	OWSTIMER (0x1DE4)	0x0854	SharePoint Foundation Upgrade	SPUpgradeSession	aj0ur	ERROR	Upgrade Timer job is exiting due to exception: Microsoft.SharePoint.Upgrade.SPUpgradeException: Upgrade completed with errors.  Review the upgrade log file located in C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\15\LOGS\Upgrade-20130503-220452-191.log.  The number of errors and warnings is listed at the end of the upgrade log file.     at Microsoft.SharePoint.Upgrade.SPUpgradeSession.CheckPoint()     at Microsoft.SharePoint.Upgrade.SPUpgradeSession.LogEnd()     at Microsoft.SharePoint.Administration.SPUpgradeJobDefinition.Execute(Guid targetInstanceId)	c401189c-5456-c0fa-03a6-f5fa06142414
```

This is due to a new method called `DoMySite_PersonalDocLicPermissionReset` in the class `Microsoft.SharePoint.Portal.UserProfiles.MySitePersonalSiteUpgrader`.  This class looks for a Document Library named "Personal Documents".  On pre-April 2013 CU, the lack of this library can cause the Cumulative Upgrade to fail with the above error.  There are two workarounds that I've identified:

* Create a document library in the user's My Site named "Personal Documents"

* Delete the user's My Site

While the April 2013 CU may be in this failed state, any My Sites created during this failed state do not suffer from the above error.