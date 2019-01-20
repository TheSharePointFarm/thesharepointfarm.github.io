---
layout: post
title: Session State Causing Upgrade Failure
tags: [sp2013]
---

This was a new one for me. I was working on a farm where the Session State Service caused a failure during the upgrade process (patching). The ULS looked like the below.

```
11/17/2016 12:20:26.13	PowerShell.exe (0x2190)	0x49A0	SharePoint Server	Session State Service	ad1m	Medium	SessionStateService(, ce4b1a37-339a-450c-9022-68fe3489c3c5) called.	fe648fa9-5071-48f0-a353-3902c7790f2b
11/17/2016 12:20:26.13	PowerShell.exe (0x2190)	0x49A0	SharePoint Server	Session State Service	ad1n	Medium	SessionStateService: Update() called	fe648fa9-5071-48f0-a353-3902c7790f2b
11/17/2016 12:20:26.13	PowerShell.exe (0x2190)	0x49A0	SharePoint Foundation	Topology	8xqz	Medium	Updating SPPersistedObject SessionStateService. Version: -1 Ensure: True, HashCode: 8275002, Id: 6bbb473f-8dbb-4d3f-b629-4bba9d391ecc, Stack:    at Microsoft.Office.Server.Administration.SessionStateService.Update()     at Microsoft.SharePoint.PowerShell.SPCmdletInstallService.InstallServiceInConfigDB(Boolean provisionTheServiceToo, String serviceRegistryKeyName)     at Microsoft.SharePoint.PowerShell.SPCmdletInstallService.InstallServices(Boolean provisionTheServicesToo)     at Microsoft.SharePoint.PowerShell.SPCmdletInstallService.InternalProcessRecord()     at Microsoft.SharePoint.PowerShell.SPCmdlet.ProcessRecord()     at System.Management.Automation.CommandProcessor.ProcessRecord()     at System.Management.Automation.CommandProcessorBase.DoExecute()     at System.Management.Automation.Internal.PipelineProcessor.SynchronousExecuteEnumerate(Object input, Hashtable errorResults, Boolean enumerate)     at System.Management.Automation.PipelineOps.InvokePipeline(Object input, Boolean ignoreInput, CommandParameterInternal[][] pipeElements, CommandBaseAst[] pipeElementAsts, CommandRedirection[][] commandRedirections, FunctionContext funcContext)     at System.Management.Automation.Interpreter.ActionCallInstruction`6.Run(InterpretedFrame frame)     at System.Management.Automation.Interpreter.EnterTryCatchFinallyInstruction.Run(InterpretedFrame frame)     at System.Management.Automation.Interpreter.EnterTryCatchFinallyInstruction.Run(InterpretedFrame frame)     at System.Management.Automation.Interpreter.Interpreter.Run(InterpretedFrame frame)     at System.Management.Automation.Interpreter.LightLambda.RunVoid1[T0](T0 arg0)     at System.Management.Automation.DlrScriptCommandProcessor.RunClause(Action`1 clause, Object dollarUnderbar, Object inputToProcess)     at System.Management.Automation.CommandProcessorBase.DoComplete()     at System.Management.Automation.Internal.PipelineProcessor.DoCompleteCore(CommandProcessorBase commandRequestingUpstreamCommandsToStop)     at System.Management.Automation.Internal.PipelineProcessor.SynchronousExecuteEnumerate(Object input, Hashtable errorResults, Boolean enumerate)     at System.Management.Automation.Runspaces.LocalPipeline.InvokeHelper()     at System.Management.Automation.Runspaces.LocalPipeline.InvokeThreadProc()     at System.Management.Automation.Runspaces.PipelineThread.WorkerProc()     at System.Threading.ExecutionContext.RunInternal(ExecutionContext executionContext, ContextCallback callback, Object state, Boolean preserveSyncCtx)     at System.Threading.ExecutionContext.Run(ExecutionContext executionContext, ContextCallback callback, Object state, Boolean preserveSyncCtx)     at System.Threading.ExecutionContext.Run(ExecutionContext executionContext, ContextCallback callback, Object state)     at System.Threading.ThreadHelper.ThreadStart()  	fe648fa9-5071-48f0-a353-3902c7790f2b
11/17/2016 12:20:26.13	PowerShell.exe (0x2190)	0x49A0	SharePoint Foundation	Upgrade	ajyxy	Medium	11/17/2016 12:20:26.13 powershell (0x2190) 0x49A0 SharePoint Foundation Upgrade SessionStateServiceSequence ajyxy INFO [SessionStateService] SchemaVersion = 14.0.1.0. fe648fa9-5071-48f0-a353-3902c7790f2b	fe648fa9-5071-48f0-a353-3902c7790f2b
11/17/2016 12:20:26.15	PowerShell.exe (0x2190)	0x49A0	SharePoint Server	Session State Service	ad1v	Medium	SessionStateService.RefreshWebConfigModificationsInternal([,b9682087-665d-4743-ab13-b6f989551b24], True) called)	fe648fa9-5071-48f0-a353-3902c7790f2b
11/17/2016 12:20:26.15	PowerShell.exe (0x2190)	0x49A0	SharePoint Server	Session State Service	ad23	Medium	SessionStateService: EnsureWebConfigModifications(svc:) called	fe648fa9-5071-48f0-a353-3902c7790f2b
11/17/2016 12:20:26.15	PowerShell.exe (0x2190)	0x49A0	SharePoint Server	Session State Service	blz4	Medium	SessionStateService: Adding HttpModules to web.config	fe648fa9-5071-48f0-a353-3902c7790f2b
11/17/2016 12:20:26.15	PowerShell.exe (0x2190)	0x49A0	SharePoint Server	Session State Service	blz5	Medium	SessionStateService: Adding SessionModule to web.config	fe648fa9-5071-48f0-a353-3902c7790f2b
11/17/2016 12:20:26.17	PowerShell.exe (0x2190)	0x49A0	SharePoint Foundation	PowerShell	6tf2	High	System.NullReferenceException: Object reference not set to an instance of an object.     at Microsoft.Office.Server.Administration.SessionStateService.GetSessionStateModification()     at Microsoft.Office.Server.Administration.SessionStateService.EnsureWebConfigModifications(SPWebService webSvc)     at Microsoft.Office.Server.Administration.SessionStateService.RefreshWebConfigModificationsInternal(SPWebService service, Boolean shouldHaveModifications)     at Microsoft.Office.Server.Administration.SessionStateService.Update()     at Microsoft.SharePoint.PowerShell.SPCmdletInstallService.InstallServiceInConfigDB(Boolean provisionTheServiceToo, String serviceRegistryKeyName)     at Microsoft.SharePoint.PowerShell.SPCmdletInstallService.InstallServices(Boolean provisionTheServicesToo)     at Microsoft.SharePoint.PowerShell.SPCmdletInstallService.InternalProcessRecord()     at Microsoft.SharePoint.PowerShell.SPCmdlet.ProcessRecord()	fe648fa9-5071-48f0-a353-3902c7790f2b
11/17/2016 12:20:26.17	PowerShell.exe (0x2190)	0x49A0	SharePoint Foundation	PowerShell	91ux	High	Error Category: InvalidData    Target Object  Microsoft.SharePoint.PowerShell.SPCmdletInstallService  Details  NULL  RecommendedAction NULL	fe648fa9-5071-48f0-a353-3902c7790f2b
```

`Install-SPService` yielded the same error. Looking at the code for the GetSessionStateModification method, it simply sets some web.config values based on property values from the Session State Service object.

```csharp
Value = string.Format(CultureInfo.InvariantCulture, "<sessionState mode=\"SQLServer\" timeout=\"{0}\" allowCustomSqlDatabase=\"true\" sqlConnectionString=\"{1}\" />", new object[] { 
    timeout.ToString(CultureInfo.InvariantCulture),
    service.Application.Database.DatabaseConnectionString
```

Basic stuff, right? So why was I getting these ULS errors? Using PowerShell, I found that Session State was enabled, which is OK.

```powershell
$farm = Get-SPFarm
$ss = $farm.Services | ?{$_.TypeName -match "session"}

Enabled Timeout  Database Server      Database Catalog     Database Id
------- -------  ---------------      ----------------     -----------
True   01:00:00
```

But you'll see that, while session state is enabled, it has no value for any of the other fields besides the timeout value. This is what caused my exception as the method, GetSessionStateModification, expected that those properties would be populated. The fix? Disable session state (or `$ss.SessionStateEnabled = $false;$ss.Update()` appended to the above PowerShell). After that, the farm upgraded just fine.