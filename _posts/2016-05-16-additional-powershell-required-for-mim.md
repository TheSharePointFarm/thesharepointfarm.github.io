---
layout: post
title: Additional PowerShell Required for Microsoft Identity Manager
tags: [mim,sp2016]
---

EDIT: This has been resolved in the February 2017 Public Update. Please review [Microsoft Identity Manager NoILMUsed Bug Fixed]({% post_url 2017-02-21-microsoft-identity-manager-noilmused-bug-fixed %}) for further information to enable the fix.

There is additional PowerShell required for Microsoft Identity Manager when using it as the identity manager for SharePoint Server 2016. Create your User Profile Service Application and enable the External Identity Manager. Once completed, run the following PowerShell from the SharePoint Management Shell.

```powershell
$sa = Get-SPServiceApplication | ?{$_.TypeName -eq 'User Profile Service Application'}
$sa.NoILMUsed = $true
$sa.Update()
```

The User Profile Service Application UI to Enable External Identity Manager does not fully work correctly (aka it's a bug). So in order to pull various properties, such as Manager, or create Audiences, make sure this is enabled. If this bug is resolved, I will update this post to reflect as such, and at that point, the additional PowerShell is not required, although you do not want to revert this setting as long as you continue to use Microsoft Identity Manager.