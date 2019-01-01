---
layout: post
title: SharePoint 2010 Farm Backup and the User Profile Service
tags: [sp2010]
---

Using the SharePoint Backup (Backup-SPFarm or via Central Administration) purposely unprovisions the User Profile Service during the backup of the User Profile Service Application.  It also automatically re-provisions itself once the backup of the User Profile Service Application is completed. Unfortunately, this means that the Farm Administrator account must be a Local Administrator on the server where the User Profile Synchronization Service is running.  This is counter to the documentation on TechNet regarding least privilege.
If the Farm Administrator account does not remain as a Local Administrator, the provisioning will fail. The failure happens during a call to the following method located in `ILMPostSetupConfiguration.cs` in the function `ConfigureMiisStage1()` in the `Microsoft.Office.Server.UserProfiles.Synchronization` assembly:

```text
ULS.SendTraceTag(0x39693230, ULSCat.msoulscat_SPS_UserProfiles, ULSTraceLevel.Medium, "ILM Configuration: Validating the system groups...");
ValidateConfigurationResult(this.validateGroupsEx(this.mmsEdition, "WSS_ADMIN_WPG", "FIMSyncOperators", "FIMSyncJoiners", "FIMSyncBrowse", "FIMSyncPasswordSet"));
```
As you can see, the service is attempting to check that the Farm Administrator is a member of the above groups. It is, by default, only a member of WSS_ADMIN_WPG. However, even after adding it to the other groups, the check fails. Unfortunately the check takes place in in the native binary, `SyncSetupUtl.dll` (located at `C:\Program Files\Microsoft Office Servers\14.0\Synchronization Service\Bin`) which cannot be decompiled to see why exactly the check is failing as a non-administrator.

What makes this issue even worse is if you have a small farm and are hosting the Central Administration site on your Application Layer along with your User Profile Synchronization Service.  This means after every backup, an `iisreset` must be scheduled after the User Profile Service is reprovisioned.

Just another note, the Farm Administrator account must be the account running the User Profile Synchronization Service.  Within ILMPostSetupConfiguration.cs, there is an explicit check to validate that this account is running the Forefront Identity Manager Synchronization Service.