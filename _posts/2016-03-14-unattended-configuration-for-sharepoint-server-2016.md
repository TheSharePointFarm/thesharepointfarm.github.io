---
layout: post
title: Unattended Configuration for SharePoint Server 2016
tags: [sp2016]
---

SharePoint Server 2016 has some... interesting things baked into it including a few things that appear to come from SharePoint Online. A new unattended configuration for SharePoint Server 2016 is also present, using an answer file. While unadvertised, it is easy to use, but a bit harder to discover as the configuration options are buried throughout the SharePoint code base.

So, how does it work? Pretty simple. Create a file called `unattend.txt`. Add some entries like this:

```
DiagnosticsULSLocation=E:\ULS
DiagnosticsLogCutInterval=1
DiagnosticsEventLogFloodProtectionEnabled=true
DiagnosticsLogMaxDiskSpaceUsageEnabled=13
DiagnosticsLogDiskSpaceUsageGB=3
```

Save it to `C:\Program Files\Common Files\microsoft shared\Web Server Extensions\16\CONFIG\` and build your farm through PowerShell or the Config Wizard. The answer file will be picked up and the values applied appropriately.

Post-farm build, you can see the key/value pairs located in the SPFarm object, which we can see via PowerShell.

```powershell
$farm = Get-SPFarm
$farm.InitializationSettings

Key                                     Value
---                                     -----
DiagnosticsULSLocation                  E:\ULS
DiagnosticsLogCutInterval               1
DiagnosticsEventLogFloodProtectionEn... true
DiagnosticsLogMaxDiskSpaceUsageEnabled  13
DiagnosticsLogDiskSpaceUsageGB          3
BrowserCEIPEnabled                      true
ErrorReportingEnabled                   true
DownloadErrorReportingUpdates           true
ClientPerformanceMeasurementEnabled     true
AppAnalyticsAutomaticUploadEnabled      true
```

There are a lot of these. Some of the keys may or may not work. This isn't a complete list of what I've found so far, and I'm compiling a spreadsheet of settings, but just examples of what is currently available.

```powershell
Property                                Type
--------                                ------
GridZoneSMTPEnableSsl                   string
GridZoneSMTPOverrideEnvelopeSender      string
GridZoneSMTPPort                        int
GridZoneSMTPServer                      string
RecycleBinRetentionPeriod               int
SecondStageRecycleBinQuota              int
TenantXP_OutgoingEmailAddress           string
TenantXP_ReplyToEmailAddress            string
UseNewsFeedEnabledSetting               bool
```

There are others, as well, such as automatically creating a Secure Store Service, etc.

Pretty neat, and hopefully this is functionality that persists into the future, as it is great for pre-configured, automated environments, versus the heavy amount of post-configuration we must use with farms today!