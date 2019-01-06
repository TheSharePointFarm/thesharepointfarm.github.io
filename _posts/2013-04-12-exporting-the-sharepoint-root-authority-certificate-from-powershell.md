---
layout: post
title: Exporting the SharePoint Root Authority Certificate from PowerShell
tags: [sp2010,sp2013]
---

If you need to export the SharePoint Root Authority Certificate from PowerShell, here is the process you'll want to follow:

```powershell
$cert = (Get-SPCertificateAuthority).RootCertificate
$ct = New-Object System.Security.Cryptography.X509Certificates.X509ContentType
$ct.value__ = 3 # 3 is for PFX, and a value of 1 is for CER
$cert.Export($ct,"Password") | Set-Content C:\export.pfx -Encoding byte # If the content type is CER, leave out the password from the export and set the file name to export.cer)
```