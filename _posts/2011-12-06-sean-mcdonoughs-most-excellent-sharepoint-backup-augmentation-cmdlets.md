---
layout: post
title: Sean McDonough’s Most Excellent SharePoint Backup Augmentation Cmdlets
tags: [sp2010]
---

Sean McDonough has introduced a project called [SharePoint Backup Augmentation Cmdlets](https://github.com/Nauplius/SharePoint-Backup-Cmdlets).  This farm-based solution adds a few new SharePoint Cmdlets to the mix:

```powershell
Get-SPBackupCatalog
Remove-SPBackupCatalog
Send-SPBackupStatus
```

The Cmdlets can be used together to automatically maintain a backup catalog, at the specified size or backup count.

To deploy the solution, copy the SharePointBAC.wsp to a local SharePoint Server.  Run the SharePoint Management Shell and navigate to the directory containing the SharePointBAC.wsp solution.  Run:

```powershell
Add-SPSolution <Full Path to SharePointBAC.wsp>
Install-SPSolution SharePointBAC.wsp -GACDeploy
```

As an example, if we wanted to keep two backups on a file share, we would run:

```powershell
$Catalog = Get-SPBackupCatalog \\fileserver\share
$Catalog | Remove-SPBackupCatalog -RetainCount 2
```

This will enumerate the existing backup catalog and then remove all but the two most recent backups.

In this example, let’s say we have a requirement where we want to keep the last backup, but only if the current backup finishes without warnings or errors:

```powershell
$Catalog = Get-SPBackupCatalog \\fileserver\share
Backup-SPFarm -Directory \\fileserver\share -BackupMethod full
$Catalog.Refresh()
if ($Catalog.LastBackupErrorCount -eq 0 -and $Catalog.LastBackupWarningCount -eq 0)
{
	$Catalog | Remove-SPBackupCatalog -RetainCount 1 -Confirm:$false
}
```

Lastly, if you want to send a status email of the last backup, we can use the Send-SPBackupStatus command with a comma-delimited list of email addresses:

```powershell
$Catalog = Get-SPBackupCatalog \\fileserver\share
$Catalog | Send-SPBackupStatus -Recipients "user1@company.com,user2@company.com"
```

If the catalog changes after you've set a variable for the catalog, do not forget to call $Catalog.Refresh()!.

These Cmdlets are at an Alpha 0.2 revision at the time of writing, however functionality-wise they have performed as expected so far.  I highly recommend these Cmdlets to anyone working with SharePoint’s built-in backup methods.