---
layout: post
title: Configuring SharePoint with PowerShell
tags: [sp2010,sp2013,sp2016]
---

Configuring SharePoint with PowerShell is quite easy, rather than running the Configuration Wizard post-installation. The primary advantage is to give you control over the Administration database. This method works with SharePoint 2010 and SharePoint 2013.

After installing SharePoint, run the SharePoint Management Shell as Administrator.

```powershell
New-SPConfigurationDatabase -DatabaseName Config -DatabaseServer SPSQL -AdministrationContentDatabase Admin -FarmCredentials (Get-Credential) -Passphrase (ConvertTo-SecureString "Password1" -AsPlainText -Force)
Initialize-SPResourceSecurity
New-SPCentralAdministration -Port 8080 -WindowsAuthProvider NTLM
Install-SPService
Install-SPFeature -AllExistingFeatures
Install-SPApplicationContent
Install-SPHelpCollection -All
```

[TechNet](http://technet.microsoft.com/en-us/library/cc262839(v=office.14).aspx) also has this information, I just had to scroll too much. And now I have a source to directly copy and paste from :-)