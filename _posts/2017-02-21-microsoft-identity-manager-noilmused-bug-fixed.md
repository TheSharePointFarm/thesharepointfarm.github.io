---
layout: post
title: Microsoft Identity Manager NoILMUsed Bug Fixed
tags: [mim,sp2016]
---

Microsoft Identity Manager required the use of the NoILMUsed property on the User Profile Service Application in SharePoint Server 2016 due to a bug that has now been fixed with the release of the February 2017 Public Update. There's a catch, though... the required timer job isn't installed due to a bug.

With the February 2017 Public Update release, a new timer job named ExternalIdentityManagerMembershipsAndRelationshipsJob is included. While it was intended to be provisioned automatically, just like with any other patch that introduced new timer jobs, that process isn't taking place. The resolution, which will hopefully be fixed for the March 2017 Public Update, is quite easy. Simply re-provision the User Profile Service Application like so:

```powershell
$sa = Get-SPServiceApplication | ?{$_.TypeName -match 'Profile'}
$sa.Provision()
```

Once this completes, validate the timer job now exists using PowerShell.

```powershell
Get-SPTimerJob | ?{$_.TypeName -match 'ExternalIdentityManagerMembershipsAndRelationshipsJob'}
```

Also remember, if you've implemented the NoILMUsed property, to revert or the new job will not run.

```powershell
$sa = Get-SPServiceApplication | ?{$_.TypeName -match 'Profile'}
$sa.NoILMUsed = $false
$sa.Update()
```

The job itself is pretty simple. It resolves the issue with the manager Profile property being blank as well as fixes the issue with Audiences not showing any membership based on Active Directory attributes. What this job does is execute two new stored procedures in the Profile database -- ImportExport_PostImportMembers and ImportExport_PostImportUserProperties.

Lastly, here is a property overview of the new timer job.

```powershell
JobDisplayName              : Updates Profile Memberships and Relationships Job
JobDescription              : Updates group membership and profile relationships with changes found in the directory.
DefaultSchedule             : every 5 minutes between 0 and 0
DisplayName                 : User Profile Service Application - Updates Profile Memberships and Relationships Job
Description                 : Updates group membership and profile relationships with changes found in the directory.
EnableBackup                : True
Service                     : UserProfileService
IsServiceJob                : True
WebApplication              :
Server                      :
LockType                    : Job
Schedule                    : every 5 minutes between 0 and 0
Title                       : User Profile Service Application - Updates Profile Memberships and Relationships Job
LastRunTime                 : 1/1/0001 12:00:00 AM
Retry                       : False
IsDisabled                  : False
VerboseTracingEnabled       : False
HistoryEntries              : {}
DiskSizeRequired            : 0
CanSelectForBackup          : False
CanRenameOnRestore          : False
CanSelectForRestore         : False
Name                        : User Profile Service Application_ExternalIdentityManagerMembershipsAndRelationshipsJob
TypeName                    : Microsoft.Office.Server.UserProfiles.OM.ExternalIdentityManagerMembershipsAndRelationshipsJob
Id                          : d0d1f00c-fd7c-4b87-9b45-b42f6901fbf4
Status                      : Online
Parent                      : UserProfileService
```