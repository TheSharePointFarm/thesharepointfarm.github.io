---
layout: post
title: Update on Incoming Email Job Lock Type Change Between SharePoint 2010 and 2013
tags: [sp2013]
---

EDIT: 12/12/2013: The `LockJobType` in SharePoint 2013 has been changed back to None in the December 2013 CU (<http://support.microsoft.com/kb/2837677/en-us>), allowing the SharePoint Administrator to run the Incoming Email Service on more than one server!

Update on the [change of the SPJobLockType]({% post_url 2013-04-16-incoming-email-service-job-lock-type-change-between-sharepoint-2010-and-2013 %}) between 2010 and 2013, the `SPJobLockType` was changed from None to Job due to a specific issue where documents may have been duplicated when there is more than one server processing Incoming Email using MX load balancing.  However, Microsoft plans to fix a release for the duplication of documents issue and revert the SharePoint 2013 SPJobLockType back to None in the December 2013 Cumulative Update.

This should restore the ability to use MX load balancing in SharePoint 2013.