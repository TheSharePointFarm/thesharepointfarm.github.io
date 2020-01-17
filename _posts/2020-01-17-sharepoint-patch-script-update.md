---
layout: post
title: SharePoint Patch Script Major Update
tags: [development,news,sp2013,sp2016,sp2019]
---

The SharePoint Patch Script is a PowerShell module that assists in installing SharePoint 2013, 2016, and 2019 patches including Cumulative Updates, hotfixes, security updates, and Public Updates.

This version is a fairly extensive re-write, adding new cmdlets: `Get-SPPatch`, `Get-SPPatchInfo`, and `Invoke-SPConfigWizard`. These cmdlets replace the previous MSI module that was provided at [SharePoint Updates](https://sharepointupdates.com) while continuing to leverage the SharePoint Updates service.

Please give the updated module a run through. You can find it on the [SharePoint Patch Script GitHub repository](https://github.com/Nauplius/SharePoint-Patch-Script). Installation and usage instructions are provided in the [wiki](https://github.com/Nauplius/SharePoint-Patch-Script/wiki), including full descriptions of each cmdlet.

Let me know your thoughts or what additions or changes you'd like to see in the GitHub Issues section!
