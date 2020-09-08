---
layout: post
title: SharePoint September 2020 Update Notes
tags: [sp2010, sp2013, sp2016, sp2019]
---

There are a couple of things I wanted to point out in this months patches for all supported versions of SharePoint.

For SharePoint 2010 and 2013, make note of the Known Issues section of specific security hotfixes (and thus the Cumulative Updates that contain them). Per Microsoft, after patching you may see the below error.

    Web Part Error: A Web Part or Web Form Control on this Page cannot be displayed or imported. The type could not be found or it is not registered as safe.

If you do encounter this (in your test environment, right?), follow the instructions on [KB4572409](https://support.microsoft.com/help/4572409) to allow your web parts/pages to render correctly.

SharePoint Server 2016 and SharePoint Server 2019 have fixes for OneDrive issues where it requires _both_ patches from the month to be applied. While you should always apply both patches from a given month (or the closest available previous month), make sure to do it this month.

SharePoint Server 2016:

[KB4484506](https://support.microsoft.com/help/4484506)

[KB4484512](https://support.microsoft.com/help/4484512)


SharePoint Server 2019:

[KB4484505](https://support.microsoft.com/help/4484505)

[KB4484504](https://support.microsoft.com/help/4484504)

There are some important security updates to consider with this months patches, namely a couple of remote code vulnerabilities that allow an authenticated attacker to execute code under the context of the IIS Application Pool or farm service (timer) account. Even though it requires authentication, consider the security of your end user accounts that do have access to SharePoint.
