---
layout: post
title: TLS 1.2 in SharePoint Server 2016 and Office Online Server
tags: [sp2016]
---

Support for TLS 1.2 in SharePoint Server 2016 and Office Online Server is available and should be one of the top considerations for securing an environment when using SSL (and you should always use SSL...). TLS 1.2 is the current version of TLS at the time of this post, and TLS 1.1 or TLS 1.2 will be required for PCI DDS (think payment systems) as of June 30th, 2018. This deadline was extended from the previous deadline of June 30th, 2016.

TLS 1.2 can be managed server-side, where it will force a client to connect over TLS 1.2, or when communicating with another server over SSL, such as Office Online Server.

For all servers, make sure [Microsoft Security Advisory 2960358](https://technet.microsoft.com/library/security/2960358) has been deployed. For systems running .NET 4.6, save the below text as enable-tls1-2.reg and import it into the server.

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319]
"SchUseStrongCrypto"=dword:00000001
```

The next step is to disable down level protocols. Again, save the below text as force-tls1-2.reg and import it into each SharePoint and OOS server.

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols]

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\PCT 1.0]
@="DefaultValue"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\PCT 1.0\Server]
@="DefaultValue"
"Enabled"=dword:00000000

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 2.0]
@="DefaultValue"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 2.0\Server]
@="DefaultValue"
"Enabled"=dword:00000000

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 3.0]
@="DefaultValue"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 3.0\Server]
@="DefaultValue"
"Enabled"=dword:00000000

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0]
@="DefaultValue"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server]
@="DefaultValue"
"Enabled"=dword:00000000

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1]
@="DefaultValue"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server]
@="DefaultValue"
"Enabled"=dword:00000000
```

Once the registry files have been imported, reboot the machine for the changes to take effect. From this point forward, the SharePoint and OOS servers will only accept connections negotiated with TLS 1.2 and reject any previous protocol version.

This is not compatible with SharePoint 2010, 2013, or Office Web Apps 2013.