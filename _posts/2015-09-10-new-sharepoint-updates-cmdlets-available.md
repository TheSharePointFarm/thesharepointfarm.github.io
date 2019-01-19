---
layout: post
title: ew SharePoint Updates Cmdlets Available
tags: [development,news]
---

The [SharePoint Updates cmdlets](http://sharepointupdates.com/PowerShell) have been updated with two fixes:

* Switches no longer require passing $true to enable. If the switch is present, it will be set to true.
* Taking into account the issue with the August 2015 Cumulative Update, a new switch named -Passive is available with Install-SPPatch. Passive mode is now disabled by default due to the issues with the August CU. This means you will be prompted to accept the EULA when the patch is executed.

Any and all feedback, as always, is welcome!