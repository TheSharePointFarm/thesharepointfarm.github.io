---
layout: post
title: Automated WAC Patch Management
tags: [oos,owa]
---

Automated WAC Patch Management is a PowerShell module to help you rebuild your Office Web Apps 2013 or Office Online Server 2016 farms in a (mostly) automated fashion. This PowerShell Module is a bit different from the previous [Automated Remote SharePoint Patch Management]({% post_url 2016-05-03-automated-remote-sharepoint-patch-management %}) script in that it is run locally on the Office Web Apps 2013 or Office Online Server 2016 servers. The [PowerShell module is available on GitHub](https://github.com/Nauplius/AWACPM).

The patch handles multiple WAC/OOS servers, rebuilding the farm exactly as you had originally configured it after patching. If you have more than one server, a new server in that farm becomes the lead host. This allows you to have minimal downtime while you patch your WAC/OOS farm. Note that if EditingEnabled is set to true, you will be prompted to confirm the setting.

For more information on usage, see the [GitHub project page](https://github.com/Nauplius/AWACPM). A [process flow diagram](https://github.com/Nauplius/AWACPM/wiki/Process-Diagram) is also available.