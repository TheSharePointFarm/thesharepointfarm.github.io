---
layout: post
title: TLS 1.2 Support for PerformancePoint Dashboard Designer
tags: [sp2016]
---

TLS 1.2 Support for PerformancePoint Dashboard Designer can only be enabled via the SchUseStrongCrypto registry entry on the client system connecting to SharePoint Server 2016. On the client system intended to run Dashboard Designer, import the following registry entry and then reboot the client system.

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319]
"SchUseStrongCrypto"=dword:00000001
```

If you enforce TLS 1.2 on SharePoint Server 2016, without this key on the client, you will receive an error when launching the Dashboard Designer.

![PPSTLSError](/assets/images/2016/05/PPSTLSError.png)