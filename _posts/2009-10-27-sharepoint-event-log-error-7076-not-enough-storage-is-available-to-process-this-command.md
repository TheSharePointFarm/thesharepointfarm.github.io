---
layout: post
title: SharePoint Event Log Error 7076 - “Not enough storage is available to process this command”
tags: [sp2007]
---

We received this message in our farm environment every 5 days after rebooting all servers within the farm.  The message appeared to be harmless, functionality-wise, but we wanted it resolved anyways.

These messages would repeat in the Application Event Log:

{% highlight text %}
Event Type: Error
Event Source: Office SharePoint Server
Event Category: Office Server Shared Services
Event ID: 7076
Date: 8/11/2008
Time: 5:08:20 AM
User: N/A
Computer: SHAREPOINT
Description:
An exception occurred while executing the Application Server Administration job.

Message: Not enough storage is available to process this command.
Techinal Support Details:
System.Runtime.InteropServices.COMException (0x80070008): Not enough storage is available to process this command.
Server stack trace:
at System.DirectoryServices.DirectoryEntry.Bind(Boolean throwIfFail)
at System.DirectoryServices.DirectoryEntry.Bind()
at System.DirectoryServices.DirectoryEntry.get_IsContainer()
at System.DirectoryServices.DirectoryEntries.CheckIsContainer()
at System.DirectoryServices.DirectoryEntries.Find(String name, String schemaClassName)
at Microsoft.SharePoint.AdministrationOperation.Metabase.MetabaseObjectCollection`1.Find(String name)
at Microsoft.SharePoint.AdministrationOperation.Metabase.MetabaseObjectCollection`1.get_Item(String name)
at Microsoft.SharePoint.AdministrationOperation.SPProvisioningAssistant.ProvisionIisApplicationPool(String name, ApplicationPoolIdentityType identityType, String userName, SecureString password, TimeSpan idleTimeout, TimeSpan periodicRestartTime)
at Microsoft.SharePoint.AdministrationOperation.SPAdministrationOperation.DoProvisionIisApplicationPool(String name, Int32 identityType, String userName, String password, TimeSpan idleTimeout, TimeSpan periodicRestartTime)
at System.Runtime.Remoting.Messaging.StackBuilderSink._PrivateProcessMessage(IntPtr md, Object[] args, Object server, Int32 methodPtr, Boolean fExecuteInContext, Object[]& outArgs)
at System.Runtime.Remoting.Messaging.StackBuilderSink.PrivateProcessMessage(RuntimeMethodHandle md, Object[] args, Object server, Int32 methodPtr, Boolean fExecuteInContext, Object[]& outArgs)
at System.Runtime.Remoting.Messaging.StackBuilderSink.SyncProcessMessage(IMessage msg, Int32 methodPtr, Boolean fExecuteInContext)

Exception rethrown at [0]:
at System.Runtime.Remoting.Proxies.RealProxy.HandleReturnMessage(IMessage reqMsg, IMessage retMsg)
at System.Runtime.Remoting.Proxies.RealProxy.PrivateInvoke(MessageData& msgData, Int32 type)
at Microsoft.SharePoint.AdministrationOperation.SPAdministrationOperation.DoProvisionIisApplicationPool(String name, Int32 identityType, String userName, String password, TimeSpan idleTimeout, TimeSpan periodicRestartTime)
at Microsoft.SharePoint.Administration.SPMetabaseManager.ProvisionIisApplicationPool(String name, Int32 identityType, String userName, SecureString password, TimeSpan idleTimeout, TimeSpan periodicRestartTime)
at Microsoft.Office.Server.Administration.SharedWebServiceInstance.CreateSharedWebServiceApplicationPool(SharedResourceProvider srp)
at Microsoft.Office.Server.Administration.ApplicationServerJob.ProvisionLocalSharedServiceInstances(Boolean isAdministrationServiceJob)
{% endhighlight %}

The resolution was simple.  Uninstall .NET 3.0 and reinstall .NET 3.0 (or if you’re reinstalling, you can also go to .NET 3.5 SP1/the latest available version).