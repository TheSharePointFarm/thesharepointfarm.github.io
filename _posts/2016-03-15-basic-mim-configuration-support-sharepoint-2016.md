---
layout: post
title: Basic MIM Configuration to Support SharePoint 2016
tags: [sp2016]
---

# Microsoft Identity Manager Series

Part 1: [Automating MIM User Profile Synchronization with SharePoint 2016]({% post_url 2016-03-09-automating-mim-user-profile-synchronization-with-sharepoint-2016 %})

Part 2: [Using MIM to Import Custom Attributes into SharePoint 2016]({% post_url 2016-03-10-using-mim-to-import-custom-attributes-into-sharepoint-2016 %})

Part 3: [Using MIM to Export Custom Attributes from SharePoint 2016]({% post_url 2016-03-11-using-mim-to-export-attributes-from-sharepoint-2016 %})

Part 4: [Default MIM to SharePoint 2016 Attribute Mappings]({% post_url 2016-03-13-default-mim-to-sharepoint-2016-attribute-mappings %})

Part 5: _Basic MIM Configuration to Support SharePoint 2016_

Part 6: [Scoping the Active Directory Management Agent in MIM]({% post_url 2016-03-16-scoping-active-directory-management-agent-mim %})

This blog post will walk you through the basic MIM configuration to support SharePoint 2016. Microsoft Identity Manager is a relatively easy installation. We will need a service account, and if you want to follow how the previous servers ran, you can of course use the Farm Administrator, however you may want to consider using a separate account due to the security we will put in place on that account.

You will need the following software:

* Microsoft Identity Manager ISO (MSDN, VLSC, etc.)
* SQL Server 2008 R2, 2012, or 2014
* SQL Server 2008 R2 or 2012 Native Client from the [SQL 2008 R2](https://www.microsoft.com/en-us/download/details.aspx?id=16978) or [2012 Feature Pack](https://www.microsoft.com/en-us/download/details.aspx?id=35580), if MIM is not running on the SQL Server
* [UserProfile.MIMSync](https://github.com/OfficeDev/PnP-Tools/tree/master/Solutions/UserProfile.MIMSync) - you can go up to the "[PnP-Tools](https://github.com/OfficeDev/PnP-Tools)" level to download a zip containing this solution
* Hotfix Rollup 4.3.2064.0 for MIM - [KB3092179](https://support.microsoft.com/en-us/kb/3092179)
* [SharePoint Connector](http://www.microsoft.com/en-us/download/details.aspx?id=41164)

## Security

Secure the Domain User service account running MIM. In the Local Security Policy, add the user to the following User Rights Assignment:

```
    Deny users access to log on as a batch job
    Deny users access to log on locally
    Deny users access to log on by using Terminal Services
    Deny users access to this computer from the network
```

## Installation

If MIM is not installed on a SQL Server, install the appropriate SQL Native Client from the Feature Pack, otherwise the MIM installation will fail. Next, validate the user running the MIM installation is a sysadmin on the SQL Server.

You will only be installing the MIM Synchronization Service. This service is provided as part of a Windows Server license. The installation bits are located on the ISO at X:\Synchronization Service\setup.exe. Start the installation.

Specify the SQL Server name and instance.

![MIM_SQLServer](/assets/images/2016/03/MIM_SQLServer.png)

Next, specify the Domain User which will run the MIM service. This is not the user who will be connecting to Active Directory and/or SharePoint.

![MIM_ServiceAcct](/assets/images/2016/03/MIM_ServiceAcct.png)

Leave the group names as default. These are local groups. Enable RPC firewall rule if Windows Firewall is enabled. Once install has completed, you will be prompted to save the encryption key for MIM. When prompted to log off and back on again, do so.

Install the MIM hotfix. The hotfix download will contain many files, but the only file we need is FIMSyncService_x64_KB3092179.msp. Open an elevated Command Prompt or PowerShell console. Use `net stop FIMSynchronizationService` or `Stop-Service FIMSynchronizationService`. This is to prevent the need to restart after the patch is installed. The service will be started automatically when the patch has completed installation.

Install the SharePoint Connector. Run SharepointConnector.msi and agree to the license terms, then complete the installation.

## Configuration

The next step is to create the Active Directory and SharePoint Management Agents via the UserProfile.MIMSync project download. Extract the contents to a folder, such as C:\SharePointSync. Using PowerShell, navigate to the folder. You will need two credentials, the synchronization account credential that has Replicate Directory Changes/All on the Domain as well as Configuration container, and the Farm Administrator account (or an account delegated Farm Administrator rights with Full Control on the UPSA as well as write rights on the Web Application where pictures are stored, if importing pictures).

First, import the module.

```powershell
Import-Module .\SharePointSync.psm1
```

In my example, s-sp2016sync is the user with Replicate Directory Changes, and s-sp2016farm is the Farm Administrator of the SharePoint 2016 farm. Nauplius.local is the DNS name of my forest, and I'm choosing to scope the Active Directory Management Agent (ADMA) to my entire domain using "DC=nauplius,DC=local", although you can scope it lower (e.g. "OU=Employees,DC=nauplius,DC=local"). My Central Administration URL is "https://centraladmin.nauplius.local" and I'm choosing to export my pictures from Active Directory to SharePoint. Note that you don't have to type all of that out, you can use the tab key to auto complete that switch.

```powershell
$syncAccount = Get-Credential -UserName "NAUPLIUS\s-sp2016sync" -Message "Sync Account"
$farmAccount = Get-Credential -UserName "NAUPLIUS\s-sp2016farm" -Message "Farm Admin"

Install-SharePointSyncConfiguration -Path C:\SharePointSync -ForestDnsName nauplius.local -ForestCredential $syncAccount -OrganizationalUnit "DC=NAUPLIUS,DC=LOCAL" -SharePointUrl https://centralAdmin.nauplius.local -SharePointCredential $farmAccount -PictureFlowDirection "Export only (NEVER from SharePoint)"
```

During installation of the Management Agents, there will be a warning that the Active Directory Management Agent password must be set prior to first run. Using the Synchronization Service application (miisclient.exe), click on the Management Agents tab. Double click on the ADMA.

![MIM_MAs](/assets/images/2016/03/MIM_MAs.png)

Click on Connect to Active Directory Forest, and enter the password for the synchronization service account. Click OK to close the ADMA.

![MIM_PasswordSet](/assets/images/2016/03/MIM_PasswordSet.png)

At this point, you're ready to start the initial full synchronization. To do so, go back to the PowerShell command prompt. In the command prompt, start the synchronization process.

```powershell
Start-SharePointSync
```

This will begin the full sync process. You will be prompted during the run if you want to continue to export the objects to SharePoint. You can bypass this by using `Start-SharePointSync -Confirm:$false`.

![MIM_MARun](/assets/images/2016/03/MIM_MARun.png)

These runs are also visible in the Synchronization Service Manager, under the Operations section. This will provide you with additional information on any warnings or errors.

That completes the installation of MIM and initial run!