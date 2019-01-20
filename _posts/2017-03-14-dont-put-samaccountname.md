---
layout: post
title: Don't put an @ in the sAMAccountName
tags: [sp2016]
---

See title. Don't put an @ in the sAMAccountName of the user. In fact, Active Directory Users and Computers will prevent you from doing so, converting the character to an underscore. But with PowerShell, we can add any symbol we want to. Here's an example:

```powershell
New-ADUser -AccountPassword (ConvertTo-SecureString 'ItsASecretToEverybody!' -AsPlainText -Force) -DisplayName 'Jane Doe' -UserPrincipalName 'janedoe@cobaltatom.com' -Name 'JaneDoe' -SamAccountName 'janedoe@contoso.com' -Enabled:$true
```

What ends up happening in this scenario is while SharePoint can resolve the username in the People Picker itself (the user's username becomes underlined), when the underlying C# code that translates usernames (which goes down to unmanaged code) does not handle an '@' symbol (and others) in the sAMAccountName. After resolving the username in the People Picker, when you hit the Share button, you'll get a 'The user does not exist or is not unique.' error. Within the ULS log, under the 'User Key' category, you'll see the following:

```
Verbose	Getting SID for windows user from AD. User: 'lab\janedoe'.
IdentityNotMappedException getting SID from AD for windows user. User: 'lab\janedoe', Exception: 'System.Security.Principal.IdentityNotMappedException: Some or all identity references could not be translated. at System.Security.Principal.NTAccount.Translate(IdentityReferenceCollection sourceAccounts, Type targetType, Boolean forceSuccess) at System.Security.Principal.NTAccount.Translate(Type targetType) at Microsoft.SharePoint.Administration.Claims.SPClaimUserKeyUtility.GetUserKeyClaimForWindows(IClaimsIdentity claimsIdentity, SPClaim userIdentityClaim)'.
```

Moral of the story is to modify the sAMAccountName to exclude the '@' symbol and other special characters disallowed by ADUC.