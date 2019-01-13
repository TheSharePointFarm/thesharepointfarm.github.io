---
layout: post
title: No Activity in User Profile Synchronization During User Sync
tags: [sp2010,sp2013]
---

I had one client where during User Profile Synchronization, profiles were not being pulled in. After they checked the `miisclient` (located in `C:\Program Files\Microsoft Office Server\15.0\Synchronization Service\UIShell`), they saw no activity on the Operations tab. It did not appear as if the Management Agent was executing. The client sent me a copy of the ULS log from the run, and I found this particular error:

```csharp
Profile sync step  (stage StartSynchronization) failed: System.Management.ManagementException: Invalid namespace     
 at System.Management.ManagementException.ThrowWithExtendedInfo(ManagementStatus errorCode)    
 at System.Management.ManagementScope.InitializeGuts(Object o)    
 at System.Management.ManagementScope.Initialize()    
 at System.Management.ManagementObjectSearcher.Initialize()    
 at System.Management.ManagementObjectSearcher.Get()    
 at Microsoft.Office.Server.UserProfiles.Synchronization.MiisServer.GetInstances(String machineName)    
 at Microsoft.Office.Server.UserProfiles.CleanupHistoryStep.Execute()    
 at Microsoft.Office.Server.UserProfiles.UserProfileImportJob.SyncJobStep.<StartExecution>b__9(Object state).
```

Looking at the method `Microsoft.Office.Server.UserProfiles.Synchronization.MiisServer.GetInstances(String machineName)`, we can see that it is attempting to query a WMI namespace:

```csharp
public static ServerCollection GetInstances(string machineName)
{
    if (string.IsNullOrEmpty(machineName))
    {
        return null;
    }
    using (ManagementObjectSearcher searcher = new ManagementObjectSearcher(new ManagementScope { Path = { NamespacePath = @"\\" + machineName + @"\root\MicrosoftIdentityIntegrationServer" }, Options = { Authentication = AuthenticationLevel.PacketPrivacy } }, new SelectQuery("MIIS_Server", null, null)))
...
}
```

Based on this information, on the server running the User Profile Synchronization Service, I had the client go to Computer Management -> Services, left click on WMI Control, then right click on WMI Control and select Properties, then go to the Advanced tab. Under Root, unlike my example here, the client's server was missing the `MicrosoftIdentityIntegrationServer` namespace:

![MicrosoftIdentityIntegrationServerNamespace](/assets/images/2013/12/MicrosoftIdentityIntegrationServerNamespace.png)

Looking again at the code, this WMI namespace is provisioned during the User Profile Synchronization Service provisioning. After having the client reprovision the UPSS (by simply stopping it and then starting it again), the User Profile Synchronization ran successfully and the `miisclient` showed activity under the Operations tab.