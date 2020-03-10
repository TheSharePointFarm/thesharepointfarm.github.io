---
layout: post
title: March 2020 Update Provides SameSite Cookie Compatibility with Chrome
tags: [sp2019]
---

The Chromium (thus Google Chrome and Microsoft Edge Chromium) introduced changes to cookie security handling. These changes known as the [SameSite Update](https://www.chromium.org/updates/same-site) may impact web applications, such as SharePoint and Office 365. While Office 365 was previously addressed by Microsoft, the March 2020 updates for SharePoint 2010 through SharePoint Server 2016 address the SameSite behavior for SharePoint on-premises. SharePoint Server 2019 was previously addressed in the February 2020 Public Update.

The updates can be found in the following KBs:

SharePoint 2010: [4484197](https://support.microsoft.com/help/4484197)

SharePoint 2013: [4484282](https://support.microsoft.com/help/4484282)

SharePoint 2016: [4484272](https://support.microsoft.com/help/4484272)

SharePoint 2019: [4484259](https://support.microsoft.com/help/4484259)

As always, I would recommend using the Cumulative Updates for SharePoint 2010 and 2013 due to previous issues when applying individual KBs (such as security-only patches). And for SharePoint Server 2016 and 2019, make sure to apply both the language independent and language dependent patches to your farm.
