---
layout: post
title: Automated Remote SharePoint Patch Management
tags: [sp2016]
---

Automated Remote SharePoint Patch Management is a PowerShell script that employes PowerShell Remoting in order to automate the process of installing a SharePoint patch across an entire farm. This requires significantly less input from an administrator then if they were to patch each host individually, regardless if they are handling it by hand or via script on each local host.

You can get the script from the [ARSPPM](https://github.com/Nauplius/ARSPPM) repo on GitHub.

This script has been tested with Windows 10 on the client and Windows Server 2012 R2 with SharePoint Server 2013 on four virtual machines. The script has two operating modes, one is to run the patches concurrently. This performs concurrent runs for each available step (with the exception of the Config Wizard, which must be run on one host at a time), speeding up the overall installation time. The second operating mode is to run each server individually, providing a form of high availability.

This PowerShell script requires that the SharePoint patch specified reside on an available Windows file share which will not be unavailable during the patching process (so don't share from one of the SharePoint servers!). Because the script uses Credssp for remoting, it should not require the client machine to be joined to the domain, although it isn't something I've personally tested and am unlikely to do so.

Here is an example usage of script.

```cscript
Import-Module .\ARSPPM.psm1
$cred = Get-Credential #must be a Local/Farm Admin on SharePoint
$ph = SP01 #This is the host where farm detection takes place, as well as where Content Databases are upgraded
$patch = "\\fileserver\patches\ubersrvprj2013-kb3114493-fullfile-x64-glb.exe" #UNC to the patch
Start-RmSPUpdate -StopServices $true -PauseSearch $true -PrimaryHost $ph -ConcurrentPatching $false -Cred $cred -PatchToApply $patch
```

There are currently a few limitations. You must be in the same directory as where ARSPPM.psm1 is located due to some hard coded pathing (we have to load the module into PowerShell jobs which requires the path). This will change in the future. The script could use additional error handling and I would greatly appreciate input via Issues on GitHub. I'd also like to hear suggestions on how to handle error handling. For example, does the script just outright stop? Or for example, if a patch fails to install on a particular host, does it patch the remainder of the hosts and just stop before it would upgrade the Content Databases and run the Config Wizard?

Let me know what you think of the script, and please provide input based on testing in your labs!