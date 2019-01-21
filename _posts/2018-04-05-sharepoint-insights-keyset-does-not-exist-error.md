---
layout: post
title: SharePoint Insights - Keyset Does Not Exist Error
tags: [office-365,sp2016]
---

I had a scenario where SharePoint Insights was throwing a "Keyset does not exist" error in the ULS log. This prevented any audit entries from being uploaded to the Office 365 Compliance Center. The errors would look similar to this, and would be present on all servers in the farm.

```csharp
Exception System.Security.Cryptography.CryptographicException: Keyset does not exist
at Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext.RunAsyncTask[T](Task`1 task)
at Microsoft.SharePoint.IdentityModel.OAuth2.SPOAuth2SecurityTokenManager.IssueESTSBearerTokenString(SPTrustedSecurityTokenService trustedProvider, String resource, String realm, IEnumerable`1 claims).
```

The source of the error was Microsoft.Office.BigData.DataLoader.exe, which is the unfriendly name for the Windows Service 'SharePoint Insights'. Working with Mike Lee, we determined the cause was that the Farm Admin account (which must be the account running SharePoint Insights), lacked permission to a particular MachineKey, though we didn't know which particular key. Mike suggested running ProcMon for which I filtered for ACCESS DENIED on FIles only. The results looked similar to this:

```
Event Class:	File System
Operation:	CreateFile
Result:	ACCESS DENIED
Path:	C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys\11111111111111111111111111111111_11111111-2222-4444-5555-666666666666
TID:	19320
Duration:	0.0000359
Desired Access:	Generic Read
Disposition:	Open
Options:	Sequential Access, Synchronous IO Non-Alert, Non-Directory File
Attributes:	n/a
ShareMode:	Read
AllocationSize:	n/a
Impersonating:	DOMAIN\FarmAdmin
```

At this point, we know exactly which file the Farm Admin is attempting to access. Fortunately the ULS tells us which certificate it is selecting prior to the ULS error. The certificate in the ULS is only identified by it's thumbprint, so we still have to correlate the thumbprint with the certificate subject name and the above path from ProcMon. I made the assumption that it was a SharePoint-generated certificate based on the ULS errors so I started with the Local Machine SharePoint store. Using some PowerShell, it easy to identify the specific certificate.

```powershell
foreach($c in gci -Path cert:\LocalMachine\SharePoint) {
    $c.Subject
    $c.Thumbprint
    $c.PrivateKey.CspKeyContainerInfo.UniqueKeyContainerName
    Write-Host
}
```

The last property is key. The `UniqueKeyContainerName` is the same value as the file name in the Path from ProcMon. We identified that it was a Security Token Service certificate. Hybrid farms have two of these certificates; only one is in use so we need to verify we're working with the correct certificate.

Unfortunately you cannot easily manage the Private Key ACL via the certificates MMC, so we'll use PowerShell to manipulate the ACL instead. I'm putting the PowerShell below in two parts. The first part will get the certificate itself and enumerate the ACLs currently in place. The second block will set our new ACL, granting WSS_RESTRICTED_WPG_V4 with Read access to the certificate.

```powershell
$thumbprint = (Get-SPServiceApplication | ?{$_.TypeName -eq 'Security Token Service Application'}).SigningCertificate.Thumbprint
$cert = gci -Path cert:\LocalMachine\SharePoint | ?{$_.ThumbPrint -eq $thumbprint}
$keys = "C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys\"
$file = $cert.PrivateKey.CspKeyContainerInfo.UniqueKeyContainerName
$acl = Get-Acl -Path ($keys+$file)
$acl.Access #outputs existing ACLs

$permission = "$($ENV:COMPUTERNAME)\WSS_RESTRICTED_WPG_V4","Read","Allow"
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule $permission
$acl.SetAccessRule($rule)
$item = Get-Item $path
$item.SetAccessControl($acl)
```

You can re-run the first block to confirm the ACL is now properly set. As soon as the ACL is set and the SharePoint Insight process runs again, you should no longer experience any errors during the upload process and begin to see audit content show up in the Office 365 Compliance Center.

This error is likely due to not running the Hybrid Configuration Wizard as the Farm Admin. In many environments, service accounts like the Farm Admin are not granted Local Administrator rights and many organizations use the best practice of denying the logon locally right SharePoint service accounts. That was the case in this particular environment, as well. Obviously it isn't idea to have to run the above fix, but ultimately it is the only choice in many environments should they leverage SharePoint Insights.

Huge thanks to Mike Lee for working with me on this!