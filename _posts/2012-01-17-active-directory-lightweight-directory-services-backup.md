---
layout: post
title: Active Directory Lightweight Directory Services – Backup
tags: [ad-lds, sp2010]
---

[Backing up AD LDS](http://technet.microsoft.com/en-us/library/cc725665(WS.10).aspx) is similar to backing up Active Directory: either take a backup of the System State via Windows Server Backup, or via the command line AD LDS utility dsdbutil.exe.

To back up an AD LDS instance via PowerShell, call dsdbutil with the required parameters.  Also note that the folder that `dsdbutil` is saving the backup to must be empty of all files and subfolders.

```PowerShell
$InstanceName = $args[0]
if($InstanceName -eq $null){exit}
$BackupRoot = "C:\backup"
ri $BackupRoot\$InstanceName\* -Confirm:$false -Recurse
dsdbutil "Activate Instance $InstanceName" ifm "Create Full $BackupRoot\$InstanceName" quit quit
```

This script can be run via `ADLDSBackup.ps1 InstanceName`.  A backup will be placed in `C:\backup\InstanceName\adamntds.dit`.