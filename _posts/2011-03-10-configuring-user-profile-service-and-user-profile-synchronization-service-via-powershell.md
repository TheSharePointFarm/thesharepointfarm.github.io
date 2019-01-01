---
layout: post
title: Configuring User Profile Service and User Profile Synchronization Service via PowerShell
tags: [sp2010]
---

Here are some scripts to configure the UPS and UPSS via PowerShell.  You'll need to fill in the blanks, of course.

```powershell
$sb = {
Add-PSSnapin Microsoft.SharePoint.PowerShell          
New-SPProfileServiceApplication -Name $args[0] -ApplicationPool $args[1] -ProfileDBName "Profile" -ProfileDBServer $args[2] -SocialDBName "Social" -SocialDBServer $args[2] -ProfileSyncDBName "Sync" -ProfileSyncDBServer $args[2] -ErrorAction SilentlyContinue -ErrorVariable er > $null
do {Start-Sleep 2} while ((Get-SPServiceApplication | where {$_.TypeName -eq "User Profile Service Application"}).Status -ne "Online")
$upa = Get-SPServiceApplication | where {$_.TypeName -eq "User Profile Service Application"}
New-SPProfileServiceApplicationProxy -Name "User Profile Service Proxy" -ServiceApplication $upa -DefaultProxyGroup > $null
} 

$upaapppool = Get-SPServiceApplicationPool -Identity "User Profile Service"
$FarmUser = Get-Credential "DOMAINFarmAccount"
$dBServer = "DatabaseServer"
$userProfileSAName = "User Profile Service"
$MySites = "http://mysites"
$job = @(Start-Job -Credential $FarmUser -ScriptBlock $sb -ArgumentList $userProfileSAName, $upaapppool.Name, $dBServer, $FarmUser.Username)| Wait-Job
$upa = Get-SPServiceApplication | where {$_.TypeName -eq "User Profile Service Application"}
Set-SPProfileServiceApplication -Identity $upa -Name $upa.Name -MySiteHostLocation $mysites
```

And for the UPSS:

```powershell
$syncMachine = Get-SPServer SharePointAppLayer
$profApp = Get-SPServiceApplication | where {$_.TypeName -eq "User Profile Service Application"}
$syncSvc = Get-SPServiceInstance -Server $syncMachine | where-object {$_.TypeName -eq "User Profile Synchronization Service"}
$syncSvc.Status = [Microsoft.SharePoint.Administration.SPObjectStatus]::Provisioning
$syncSvc.IsProvisioned = $false
$syncSvc.UserProfileApplicationGuid = $profApp.Id
$syncSvc.Update()

$profApp.SetSynchronizationMachine($syncMachine.Address, $syncSvc.Id, “DOMAINFarmAcctount”, “Password”)
if ($syncSvc.Status -ne "Online") {
Start-SPServiceInstance $syncSvc | out-null
}

do {Start-Sleep 2} while ((Get-SPServiceInstance -Server $syncMachine | where {$_.TypeName -eq "User Profile Synchronization Service"}).Status -ne "Online")
```

If you need to mask the password for the UPSS, you can use the below function.  You will need to perform a Get-Credential on the Farm User account, e.g.:

```powershell
$FarmUser = Get-Credential "DOMAINFarmAccount"
```

Change the line `$profApp.SetSynchronizationMachine` to:

```powershell
$profApp.SetSynchronizationMachine($syncMachine.Address, $syncSvc.Id, $FarmUser.UserName, (ConvertTo-UnsecureString $FarmUser.Password))
```

Then add this function:

```powershell
Function ConvertTo-UnsecureString([System.Security.SecureString]$string)
{
	$unmanagedString = [System.Runtime.InteropServices.Marshal]::SecureStringToGlobalAllocUnicode($string)
	$unsecureString = [System.Runtime.InteropServices.Marshal]::PtrToStringUni($unmanagedString)
	System.Runtime.InteropServices.Marshal]::ZeroFreeGlobalAllocUnicode($unmanagedString)
	return $unsecureString
}
```