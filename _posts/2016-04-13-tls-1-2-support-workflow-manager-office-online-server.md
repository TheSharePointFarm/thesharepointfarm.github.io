---
layout: post
title: TLS 1.2 Support for Workflow Manager and Office Online Server
tags: [oos,sp2016,wfm]
---

TLS 1.2 support for Workflow Manager and Office Online Server when communicating with a SharePoint Server 2016 farm that has forced TLS 1.2 communications must be manually enabled when Office Online Server or Workflow Manager are installed on Windows Server 2012 R2. There are two requirements to enable TLS 1.2 support. First, make sure [Update for Disabling RC4 in .NET TLS](https://technet.microsoft.com/en-us/library/security/2960358.aspx) is installed. Second, add the SchUseStrongCrypto value.

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319]
"SchUseStrongCrypto"=dword:00000001
```

Import the key into each OOS/WFM server, then restart. If SharePoint Server 2016 has enforced TLS 1.2, you will run into socket connection forcibly closed messages on OOS/WFM without this key.

This key does require a reboot of the server. Once that is complete, connections with OOS and WFM will succeed.