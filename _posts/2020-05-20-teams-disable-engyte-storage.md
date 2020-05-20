---
layout: post
title: Disable Engyte Cloud Storage Prior to Feature Rollout for Microsoft Teams
tags: [teams]
---

Microsoft is continuing to add 3rd party storage providers to Microsoft Teams, with the next one being Egnyte which will rollout to all customers in mid-June as part of [63706](https://www.microsoft.com/microsoft-365/roadmap?filters=&searchterms=63706). Microsoft is _enabling_ this storage provider by default. If your organization implements data policies which allow for only certain storage providers to be leveraged in your environment, this is problematic -- since features rollout gradually, there is no specific date in which an administrator can turn this off from the Teams Admin Center.

There is a workaround, however. You can disable it via PowerShell today using the `SkypeOnlineConnector` module. [Download and install](https://docs.microsoft.com/skypeforbusiness/set-up-your-computer-for-windows-powershell/download-and-install-the-skype-for-business-online-connector) the SfB Online module, then run the following PowerShell cmdlets.

Accounts without MFA enabled:

```powershell
Import-Module SkypeOnlineConnector
$cred = Get-Credential
$sess = New-CsOnlineSession -Credential $cred
Import-PSSession $sess
Set-CsTeamsClientConfiguration -AllowEgnyte $false
```

Accounts with MFA enabled:

```powershell
Import-Module SkypeOnlineConnector
$sess = New-CsOnlineSession -UserName admin@contoso.com
Import-PSSession $sess
Set-CsTeamsClientConfiguration -AllowEgnyte $false
```

Once set, Engyte will be disabled by default when the feature rolls out to your tenant.
