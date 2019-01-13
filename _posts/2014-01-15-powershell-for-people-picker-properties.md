---
layout: post
title: PowerShell for People Picker Properties
tags: [sp2010,sp2013]
---

As many SharePoint Administrators are aware, especially those dealing with one-way trusts or Selective trusts, the `peoplepicker-*` properties are very familiar to us. Since at least WSSv3, if not earlier, the `peoplepicker-*` properties have only been available via `stsadm` with no direct PowerShell replacement (it could be done, but it isn't as pretty as 'native' PowerShell).

For SharePoint 2010 and 2013, however, easier PowerShell-accessible properties were put into place. This allows the SharePoint Administrator to quickly configure these in a much more 'modern' way. Let's say I need to configure the People Picker on a Web Application to filter out Groups. The classic way I would do this is:

```powershell
stsadm -o setproperty -pn peoplepicker-searchadcustomquery -pv "(&(objectCategory=person)(objectClass=user))" -url http://webAppUrl
```

This property can be retrieved with `stsadm -o getproperty -pn peoplepicker-searchadcustomquery -url http://webAppUrl`. While you can still set the People Picker properties this way, the nice PowerShell way of doing it moving forward, taking the previous example, is:

```powershell
$wa = Get-SPWebApplication http://webAppUrl
$wa.PeoplePickerSettings.ActiveDirectoryCustomQuery = "(&(objectCategory=person)(objectClass=user))"
$wa.Update()
```

To remove it, we simply nullify it:

```powershell
$wa = Get-SPWebApplication http://spwebapp1
$wa.PeoplePickerSettings.ActiveDirectoryCustomFilter = $null
$wa.Update()
```

This works with all of the properties displayed by `(Get-SPWebApplication http://spwebapp1).PeoplePickerSettings` except for `SearchActiveDirectoryDomains` (previous `peoplepicker-searchadforests`). For this particular property, we have to take some extra steps. First, use PowerShell to set the application credential key:

```powershell
$key = ConvertTo-SecureString "Password1" -AsPlainText -Force
[Microsoft.SharePoint.SPSecurity]::SetApplicationCredentialKey($key)
```

This will set the key used to encrypt the credentials of the password we set for the user in the SearchActiveDirectoryDomains connection. Next, setup the connection:

```powershell
$wa = Get-SPWebApplication http://webAppUrl
$adsearchobj = New-Object Microsoft.SharePoint.Administration.SPPeoplePickerSearchActiveDirectoryDomain
$userpassword = ConvertTo-SecureString "UserPassword1" -AsPlainText -Force #Password for the user account CONTOSO\s-useraccount
$adsearchobj.DomainName = "contoso.com"
$adsearchobj.ShortDomainName = "CONTOSO" #Optional
$adsearchobj.IsForest = $true #$true for Forest, $false for Domain
$adsearchobj.LoginName = "s-useraccount"
$adsearchobj.SetPassword($userpassword)

$wa.PeoplePickerSettings.SearchActiveDirectoryDomains.Add($adsearchobj)
$wa.Update()
```

If you need to search multiple domains (or forests), just create more of the SPPeoplePickerSearchActiveDirectoryDomain objects and add them to `$wa.PeoplePickerSettings.SearchActiveDirectoryDomains`. To revert the changes, you can either clear all entries via `$wa.PeoplePickerSettings.SearchActiveDirectoryDomains.Clear(); $wa.Update()`, a specific entry by using the zero-based index, like so `$wa.PeoplePickerSettings.SearchActiveDirectoryDomains.RemoveAt(0)` or alternatively, we can retrieve a specific entry by retrieving it, then removing it, like so:

```powershell
$adsearchobj = $wa.PeoplePickerSettings.SearchActiveDirectoryDomains.Item(0) #zero-based index
$wa.PeoplePickerSettings.SearchActiveDirectoryDomains.Remove($adsearchobj)
$wa.Update()
```

Unfortunately, TechNet, even for SharePoint 2013, still leads us down the way of using stsadm to set these People Picker Properties. But hopefully this gives you insight on how to do it in a more 'modern' way, to help you further retire the use of stsadm.