---
layout: post
title: Replacing stsadm -o sync with PowerShell
tags: [sp2013,sp2016]
---

One of the only `stsadm.exe` commands I use today is `stsadm -o sync`. I thought it would be a great idea to replace this 'final' stsadm command with PowerShell. I will be covering two equivalent cmdlets, one to cover `stsadm -o sync -listolddatabases 0` and another to cover `stsadm -o sync -deleteolddatabases 0`. What these specific commands do is clear the last sweep time that the User Profile Service Application timer jobs pushed User Profile Information from the Profile database into each Content Database for use within the UserInfo lists on Site Collections. The `-listolddatabases 0` is part of the troubleshooting process to see when this push last ran successfully. Typically if it is older than one day, you would use `-deleteolddatabases 0` to clear out that synchronization information. The next time the UPSA -> UIL timer jobs ran, the UserInfo data in Site Collections would be up-to-date.

This module provides two cmdlets:

```powershell
Get-SPDatabaseSyncInformation
Clear-SPDatabaseSyncInformation
```

Both parameters allow you to pass an integer representing the number of days. If no value is passed, 0 will be used (which will most likely report all databases). The script also validates that there is an available User Profile Service Application Proxy in the farm, as well as attached to the Web Application. Content Databases in Web Applications that do not have a UPSA Proxy will be skipped. Save the below script as `SharePointDatabaseSync.psm1`. Use `Import-Module` to import it into the current PowerShell session, or place the script in `C:\Users\username\Documents\WindowsPowerShell\Modules\SharePointDatabaseSync\`. This will allow the module to automatically load on Windows Server 2012 or higher.

```powershell
Add-PSSnapin Microsoft.SharePoint.PowerShell -EA 0

function Get-SPUPSAProxy
{
    $proxy = Get-SPServiceApplicationProxy | ?{$_.TypeName -match 'User Profile Service Application Proxy'} | Select -First 1

    if ($proxy -eq $null)
    {
        Write-Host -ForegroundColor Red 'No User Profile Service Application Proxy available.'
        break
    }
    else
    {
        Write-Host -ForegroundColor Yellow "Found proxy $($proxy.Name.ToString())`n"
        return $proxy
    }
}

function Get-SPDatabaseSyncInformation
{
   param
   (
     [int]
     [Parameter(Mandatory=$false)]
     $Days = 0
   )

    $proxy = Get-SPUPSAProxy
    $now = (Get-Date).AddDays(-$Days)
    $i = 0

    foreach($db in Get-SPContentDatabase)
    {
        $proxyGroup = $db.WebApplication.ServiceApplicationProxyGroup
        
        if($proxyGroup.ContainsType($proxy.GetType()))
        {
            if ($db.LastProfileSyncTime.ToLocalTime() -lt $now)
            {
                ++$i
                Write-Host -ForegroundColor Green "`t$($db.Name.ToString()):"
                Write-Host -ForegroundColor Yellow "`t`tLast Synchronized: $($db.LastProfileSyncTime.ToLocalTime())"
            }
        }
    }
    Write-Host -ForegroundColor Green "`n$i databases exceeding profile synchronization time."
}

function Clear-SPDatabaseSyncInformation
{
    param
    (
        [int]
        [Parameter(Mandatory=$false)]
        $Days = 0
    )
    $proxy = Get-SPUPSAProxy
    $now = (Get-Date).AddDays(-$Days)
    $i = 0

    foreach($db in Get-SPContentDatabase)
    {
        $proxyGroup = $db.WebApplication.ServiceApplicationProxyGroup
        
        if($proxyGroup.ContainsType($proxy.GetType()))
        {
            if ($db.LastProfileSyncTime.ToLocalTime() -lt $now)
            {
                ++$i
                Write-Host -ForegroundColor Green "Clearing Sync Data for $($db.Name.ToString())"
                [Microsoft.Office.Server.UserProfiles.WSSProfileSynch]::ClearSyncDataForContentDatabase($db)
            }
        }
    }
    Write-Host -ForegroundColor Green "`nCompleted clearing synchronization information for $i databases."
}
```

Ultimately, the output will look similar to the pictures below.

![DatabaseSyncInfo](/assets/images/2016/04/DatabaseSyncInfo.png)

If you have any thoughts or possible improvements, let me know.