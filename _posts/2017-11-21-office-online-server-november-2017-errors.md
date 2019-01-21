---
layout: post
title: Office Online Server November 2017 Errors
tags: [oos]
---

Office Online Server was refreshed as a new release available from MSDN or the Volume License Center in November 2017. After installing the new binary, a couple of errors may occur. I've reproduced this on multiple farms.

The first, and correctable error is with the `RtcWatchdog.exe` service. You may see similar errors to this in the Application Event Log as well as ULS log.

```csharp
An exception occurred during a watchdog callback for Rtc2Watchdog. System.IO.FileNotFoundException: Could not load file or assembly 'Newtonsoft.Json, Version=4.5.0.0, Culture=neutral, PublicKeyToken=30ad4fe6b2a6aeed' or one of its dependencies. The system cannot find the file specified.  File name: 'Newtonsoft.Json, Version=4.5.0.0, Culture=neutral, PublicKeyToken=30ad4fe6b2a6aeed'    
 at Microsoft.AspNet.SignalR.Client.Connection..ctor(String url, String queryString)    
 at Microsoft.AspNet.SignalR.Client.HubConnection..ctor(String url, Boolean useDefaultUrl)    
 at Microsoft.Office.Web.RtcWatchdog.RtcWatchdog.HubConnectionFromUri(String uri, HubConnection& connection, IHubProxy& echo)    
 at Microsoft.Office.Web.RtcWatchdog.RtcWatchdog.GetRtc2AgentHealthStatus(String rtcUri, String& message)    
 at Microsoft.Office.Web.RtcWatchdog.RtcWatchdog.CheckServiceInstance(ServiceInstance instance)    
 at Microsoft.Office.Web.Common.WatchdogHelperThreadManager.GetHealthResults(WatchdogExecutionContext context, ServiceInstance si)    WRN: Assembly binding logging is turned OFF.  To enable assembly bind failure logging, set the registry value [HKLM\Software\Microsoft\Fusion!EnableLog] (DWORD) to 1.  Note: There is some performance penalty associated with assembly bind failure logging.  To turn this feature off, remove the registry value [HKLM\Software\Microsoft\Fusion!EnableLog].   StackTrace: 
 at uls.native.dll: (sig=93921b0b-1b34-4c76-b38c-fe12f5638234|2|uls.native.pdb, offset=29BE5)
 at uls.native.dll: (offset=1F9BE)
```

This is due to the binary, Microsoft.AspNet.SignalR.Client.dll looking for the binary `Newtonsoft.Json.dll`. The DLL is provisioned in the GAC, however it isn't the version that the `SignalR.Client.dll` expects. SignalR is looking for version 4.5.0.0 of the binary while the version 9.0.1.19813 is installed by the November 2017 OOS installer. The fix for this is simple, we perform a binary redirect. In the file `C:\Program Files\Microsoft Office Web Apps\Rtc2Watchdog\Rtc2Watchdog.exe.config`, add the following lines before the closing `</configuration>` element.

```xml
<runtime>
    <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
       <dependentAssembly>
           <assemblyIdentity name="Newtonsoft.Json"
               publicKeyToken="30AD4FE6B2A6AEED" culture="neutral"/>
           <bindingRedirect oldVersion="0.0.0.0-6.0.0.0" newVersion="9.0.0.0"/>
       </dependentAssembly>
    </assemblyBinding>
</runtime>
```

You do not need to restart any services after making this change.

The second error is with `UlsFileWriterWatchdog.exe`. This particular error I do not believe is correctable without a patch from Microsoft, but what you'll see in the ULS logs is similar to this:

```csharp
Unhandled exception is being reported by SI platform. Microsoft.Office.ServiceInfrastructure.Runtime.Settings.SettingNotDefinedException: Cluster specific setting 'MonitoringAgentUlsFileWriterEnabled' not defined or of the wrong type.    
 at Microsoft.Office.ServiceInfrastructure.Runtime.Settings.OsiClusterSpecificSetting`1.get_CurrentValue()    
 at Microsoft.Office.ServiceInfrastructure.Telemetry.Monitoring.UlsFileWriterWatchdogChecks.ServiceSetup()    
 at Microsoft.Office.ServiceInfrastructure.Telemetry.Monitoring.UlsFileWriterWatchdog.Main(String[] args) Microsoft.Office.ServiceInfrastructure.Runtime.Settings.SettingNotDefinedException: Cluster specific setting 'MonitoringAgentUlsFileWriterEnabled' not defined or of the wrong type.    
 at Microsoft.Office.ServiceInfrastructure.Runtime.Settings.OsiClusterSpecificSetting`1.get_CurrentValue()    
 at Microsoft.Office.ServiceInfrastructure.Telemetry.Monitoring.UlsFileWriterWatchdogChecks.ServiceSetup()    
 at Microsoft.Office.ServiceInfrastructure.Telemetry.Monitoring.UlsFileWriterWatchdog.Main(String[] args) StackTrace: 
 at uls.native.dll: (sig=93921b0b-1b34-4c76-b38c-fe12f5638234|2|uls.native.pdb, offset=29BE5)
 at uls.native.dll: (offset=1F9BE)
```

Neither of these appear to have an end-user impact, but they will flood the ULS as well as Application Event Logs every few seconds.