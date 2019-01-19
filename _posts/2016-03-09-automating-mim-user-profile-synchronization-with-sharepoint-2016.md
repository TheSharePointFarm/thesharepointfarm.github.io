---
layout: post
title: Automating MIM User Profile Synchronization with SharePoint 2016
tags: [sp2016,mim]
---

# Microsoft Identity Manager Series

Part 1: _Automating MIM User Profile Synchronization with SharePoint 2016_

Part 2: [Using MIM to Import Custom Attributes into SharePoint 2016]({% post_url 2016-03-10-using-mim-to-import-custom-attributes-into-sharepoint-2016 %})

Part 3: [Using MIM to Export Custom Attributes from SharePoint 2016]({% post_url 2016-03-11-using-mim-to-export-attributes-from-sharepoint-2016 %})

Part 4: [Default MIM to SharePoint 2016 Attribute Mappings]({% post_url 2016-03-13-default-mim-to-sharepoint-2016-attribute-mappings %})

Part 5: [Basic MIM Configuration to Support SharePoint 2016]({% post_url 2016-03-15-basic-mim-configuration-support-sharepoint-2016 %})

Part 6: [Scoping the Active Directory Management Agent in MIM]({% post_url 2016-03-16-scoping-active-directory-management-agent-mim %})

In order to do the fancy stuff we used to do with the User Profile Synchronization Service, such as import User Profile Photos from the thumbnailPhoto attribute or export attributes from SharePoint to Active Directory, SharePoint 2016 requires Microsoft Identity Manager. It's a fairly simple setup, and the scripts to create the Management Agents are available from the [Office PnP solutions](https://github.com/OfficeDev/PnP-Tools/tree/master/Solutions/UserProfile.MIMSync). What is missing is automating MIM User Profile Synchronization with SharePoint 2016.

To have a continual synchronization, you'll need to set up a couple of Scheduled Tasks on the MIM server. We have two scripts, one for a Full Synchronization and one for a Delta Synchronization.

The first one is the Full Synchronization script named `MIMSharePointSyncTask.ps1` and located at `C:\SharePointSync\`.

```powershell
Import-Module "C:\SharePointSync\SharePointSync.psm1"
Start-SharePointSync -Confirm:$false
```

The second is a Delta Synchronization script named `MIMSharePointSyncDelta.ps1` and located in the same directory, `C:\SharePointSync\`.

```powershell
Import-Module "C:\SharePointSync\SharePointSync.psm1"
Start-SharePointSync -Delta -Confirm:$false
```

Next, we have two Scheduled Tasks, again one for a full synchronization and one for a delta synchronization. These two Tasks have been exported to XML and can be imported through the Task Scheduler graphic interface. Replace "DOMAIN\USERNAME" located in two places, in the <Author> and <UserId> elements, with a valid domain\username value. If the MIM files (that is, SharePointSync.psm1) are not located at C:\SharePointSync, you will need to manually fix that up as well. The full synchronization runs on Sunday, while the delta synchronization runs every 30 minutes.

Note that you can simply import the XML file and then make the changes through the graphical interface, if that is more comfortable for you.

Full Synchronization Scheduled Task:

```xml
<?xml version="1.0" encoding="UTF-16"?>
<Task version="1.4" xmlns="http://schemas.microsoft.com/windows/2004/02/mit/task">
  <RegistrationInfo>
    <Date>2016-02-22T21:20:16.7543893</Date>
    <Author>DOMAIN\USERNAME</Author>
  </RegistrationInfo>
  <Triggers>
    <CalendarTrigger>
      <StartBoundary>2016-02-22T21:19:15.6761523</StartBoundary>
      <Enabled>true</Enabled>
      <ScheduleByWeek>
        <DaysOfWeek>
          <Sunday />
        </DaysOfWeek>
        <WeeksInterval>1</WeeksInterval>
      </ScheduleByWeek>
    </CalendarTrigger>
  </Triggers>
  <Principals>
    <Principal id="Author">
      <UserId>DOMAIN\USERNAME</UserId>
      <LogonType>S4U</LogonType>
      <RunLevel>HighestAvailable</RunLevel>
    </Principal>
  </Principals>
  <Settings>
    <MultipleInstancesPolicy>IgnoreNew</MultipleInstancesPolicy>
    <DisallowStartIfOnBatteries>true</DisallowStartIfOnBatteries>
    <StopIfGoingOnBatteries>true</StopIfGoingOnBatteries>
    <AllowHardTerminate>true</AllowHardTerminate>
    <StartWhenAvailable>false</StartWhenAvailable>
    <RunOnlyIfNetworkAvailable>false</RunOnlyIfNetworkAvailable>
    <IdleSettings>
      <StopOnIdleEnd>true</StopOnIdleEnd>
      <RestartOnIdle>false</RestartOnIdle>
    </IdleSettings>
    <AllowStartOnDemand>true</AllowStartOnDemand>
    <Enabled>true</Enabled>
    <Hidden>false</Hidden>
    <RunOnlyIfIdle>false</RunOnlyIfIdle>
    <DisallowStartOnRemoteAppSession>false</DisallowStartOnRemoteAppSession>
    <UseUnifiedSchedulingEngine>false</UseUnifiedSchedulingEngine>
    <WakeToRun>false</WakeToRun>
    <ExecutionTimeLimit>P3D</ExecutionTimeLimit>
    <Priority>7</Priority>
  </Settings>
  <Actions Context="Author">
    <Exec>
      <Command>powershell.exe</Command>
      <Arguments>C:\SharePointSync\MIMSharePointSyncTask.ps1</Arguments>
      <WorkingDirectory>C:\SharePointSync</WorkingDirectory>
    </Exec>
  </Actions>
</Task>
```

And here is the MIM Delta Synchronization Scheduled Task:

```xml
<?xml version="1.0" encoding="UTF-16"?>
<Task version="1.4" xmlns="http://schemas.microsoft.com/windows/2004/02/mit/task">
  <RegistrationInfo>
    <Date>2016-02-22T21:28:13.6363094</Date>
    <Author>DOMAIN\USERNAME</Author>
  </RegistrationInfo>
  <Triggers>
    <CalendarTrigger>
      <Repetition>
        <Interval>PT30M</Interval>
        <Duration>P1D</Duration>
        <StopAtDurationEnd>false</StopAtDurationEnd>
      </Repetition>
      <StartBoundary>2016-02-22T21:27:24.2225148</StartBoundary>
      <Enabled>true</Enabled>
      <ScheduleByDay>
        <DaysInterval>1</DaysInterval>
      </ScheduleByDay>
    </CalendarTrigger>
  </Triggers>
  <Principals>
    <Principal id="Author">
      <UserId>DOMAIN\USERNAME</UserId>
      <LogonType>S4U</LogonType>
      <RunLevel>HighestAvailable</RunLevel>
    </Principal>
  </Principals>
  <Settings>
    <MultipleInstancesPolicy>IgnoreNew</MultipleInstancesPolicy>
    <DisallowStartIfOnBatteries>true</DisallowStartIfOnBatteries>
    <StopIfGoingOnBatteries>true</StopIfGoingOnBatteries>
    <AllowHardTerminate>true</AllowHardTerminate>
    <StartWhenAvailable>false</StartWhenAvailable>
    <RunOnlyIfNetworkAvailable>false</RunOnlyIfNetworkAvailable>
    <IdleSettings>
      <StopOnIdleEnd>true</StopOnIdleEnd>
      <RestartOnIdle>false</RestartOnIdle>
    </IdleSettings>
    <AllowStartOnDemand>true</AllowStartOnDemand>
    <Enabled>true</Enabled>
    <Hidden>false</Hidden>
    <RunOnlyIfIdle>false</RunOnlyIfIdle>
    <DisallowStartOnRemoteAppSession>false</DisallowStartOnRemoteAppSession>
    <UseUnifiedSchedulingEngine>false</UseUnifiedSchedulingEngine>
    <WakeToRun>false</WakeToRun>
    <ExecutionTimeLimit>P1D</ExecutionTimeLimit>
    <Priority>7</Priority>
  </Settings>
  <Actions Context="Author">
    <Exec>
      <Command>PowerShell</Command>
      <Arguments>"C:\SharePointSync\MIMSharePointSyncDelta.ps1"</Arguments>
      <WorkingDirectory>C:\SharePointSync</WorkingDirectory>
    </Exec>
  </Actions>
</Task>
```

Once the Tasks have successfully run, do not forget to run another task on a SharePoint server to update the user profile photos. You will need to store the password for this account for the Task, and this account will need to be able to write to the MySite Host as well as have Full Control over the User Profile Service Application. The script is integrated into the arguments for this Task, and this Task runs at 11 PM every day. Adjust as you see fit. Do not forget to update the `<Arguments>`element, specifically the value of the `-MySiteHostLocation`, otherwise this will not work.

```xml
<?xml version="1.0" encoding="UTF-16"?>
<Task version="1.4" xmlns="http://schemas.microsoft.com/windows/2004/02/mit/task">
  <RegistrationInfo>
    <Date>2016-03-09T11:48:43.4842001</Date>
    <Author>DOMAIN\USERNAME</Author>
  </RegistrationInfo>
  <Triggers>
    <CalendarTrigger>
      <StartBoundary>2016-03-09T23:00:00</StartBoundary>
      <Enabled>true</Enabled>
      <ScheduleByDay>
        <DaysInterval>1</DaysInterval>
      </ScheduleByDay>
    </CalendarTrigger>
  </Triggers>
  <Principals>
    <Principal id="Author">
      <UserId>DOMAIN\USERNAME</UserId>
      <LogonType>Password</LogonType>
      <RunLevel>HighestAvailable</RunLevel>
    </Principal>
  </Principals>
  <Settings>
    <MultipleInstancesPolicy>IgnoreNew</MultipleInstancesPolicy>
    <DisallowStartIfOnBatteries>true</DisallowStartIfOnBatteries>
    <StopIfGoingOnBatteries>true</StopIfGoingOnBatteries>
    <AllowHardTerminate>true</AllowHardTerminate>
    <StartWhenAvailable>false</StartWhenAvailable>
    <RunOnlyIfNetworkAvailable>false</RunOnlyIfNetworkAvailable>
    <IdleSettings>
      <StopOnIdleEnd>true</StopOnIdleEnd>
      <RestartOnIdle>false</RestartOnIdle>
    </IdleSettings>
    <AllowStartOnDemand>true</AllowStartOnDemand>
    <Enabled>true</Enabled>
    <Hidden>false</Hidden>
    <RunOnlyIfIdle>false</RunOnlyIfIdle>
    <DisallowStartOnRemoteAppSession>false</DisallowStartOnRemoteAppSession>
    <UseUnifiedSchedulingEngine>false</UseUnifiedSchedulingEngine>
    <WakeToRun>false</WakeToRun>
    <ExecutionTimeLimit>PT12H</ExecutionTimeLimit>
    <Priority>7</Priority>
  </Settings>
  <Actions Context="Author">
    <Exec>
      <Command>powershell.exe</Command>
      <Arguments>Add-PSSnapIn Microsoft.SharePoint.PowerShell -EA 0;Update-SPProfilePhotoStore -MySiteHostLocation https://mySiteHost.domain.com -CreateThumbnailsForImportedPhotos 1</Arguments>
    </Exec>
  </Actions>
</Task>
```

Once those scripts have been set up, the MIM synchronization and generation of User Profile Photos is complete!