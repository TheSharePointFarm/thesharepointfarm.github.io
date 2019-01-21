---
layout: post
title: KB4011021 May Fail to Install for Office Online Server
tags: [oos]
---

EDIT: 1/18/2018: This update has been replaced by [KB4011022](https://support.microsoft.com/help/4011022).

[KB4011021](https://support.microsoft.com/en-us/help/4011021/descriptionofthesecurityupdateforofficeonlineserverjanuary) may fail to install for Office Online Server. This appears to be due to a failure to install the GAC binaries. To prevent a partial installation of this KB, I would suggest holding off on applying this to production systems. The wacserver-x-none_MSPLOG.LOG installation log file will have entries similar to the below.

```
01/11/2018 12:33:39.445 [12820]: Assembly Install: Failing with hr=80070005 at RemoveDirectoryAndChildren, line 393
01/11/2018 12:33:39.445 [12820]: Detailed info about C:\Windows\assembly\temp\SP0ZKA0QET\Microsoft.Office.Web.Apps.Environment.WacServer.intl.dll
01/11/2018 12:33:39.445 [12820]: File attributes: 00000080
01/11/2018 12:33:39.476 [12820]: Restart Manager Info: 1 entries
01/11/2018 12:33:39.476 [12820]: App[0]: (10644) Windows PowerShell (), type = 5
01/11/2018 12:33:39.476 [12820]: Security info:
01/11/2018 12:33:39.476 [12820]: Owner: S-1-5-18
01/11/2018 12:33:39.476 [12820]: Group: S-1-5-18
01/11/2018 12:33:39.476 [12820]: DACL information: 5 entries:
01/11/2018 12:33:39.476 [12820]: ACE[0]: Type = 0x00, Flags = 010, Mask = 001f01ff, SID = S-1-5-18
01/11/2018 12:33:39.476 [12820]: ACE[1]: Type = 0x00, Flags = 010, Mask = 001f01ff, SID = S-1-5-32-544
01/11/2018 12:33:39.476 [12820]: ACE[2]: Type = 0x00, Flags = 010, Mask = 001200a9, SID = S-1-5-32-545
01/11/2018 12:33:39.476 [12820]: ACE[3]: Type = 0x00, Flags = 010, Mask = 001200a9, SID = S-1-15-2-1
01/11/2018 12:33:39.476 [12820]: ACE[4]: Type = 0x00, Flags = 010, Mask = 001200a9, SID = S-1-15-2-2
```

If you have attempted to install and the installation failed, re-install the November 2017 ISO and subsequently install [KB4011020](https://support.microsoft.com/en-us/help/4011020/description-of-the-security-update-for-office-online-server-november-2).