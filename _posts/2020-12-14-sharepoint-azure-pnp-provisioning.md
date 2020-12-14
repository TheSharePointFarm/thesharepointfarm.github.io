---
layout: post
title: SharePoint PnP Provisioning with Azure Logic Apps and Azure Automation
tags: [spo,teams]
---

This guide will walk you through provisioning of SharePoint Online modern sites using the Graph API for permissions. The overall process is:

* Export PnP template
* Create Azure Blob Storage account with a File Share containing the exported template
* Create an Azure Automation account
* Configure Graph API permission; create Client Secret (though we'll be primarily using the Client Certificate)
* Create an Azure Runbook (PowerShell)
* Create and configure an Azure Logic App
* Create a new SharePoint Online Site Design
* Test!

## Export PnP Template

We'll start with an existing SharePoint Online Team site (this also applies to Communication sites, however there are some extra settings specifically for Team sites that I want to point out). Set up the site with the desired configuration including Lists, Libraries, and so on.

The first step is to export your site as a PnP package.

```powershell
Connect-PnPOnline -Url https://cobaltatom.sharepoint.com/sites/templatesite -PnPManagementShell
Get-PnPProvisioningTemplate -Out template.xml
```

## Create Azure Blob Storage account

Navigate to the [Azure Portal](https://portal.azure.com) and create a new Resource Group. All objects we'll be creating should be closest to your SharePoint Online data center, i.e. if you provisioned your M365 tenant in the western United States, use West US or West US 2 (West US 2 is generally slightly cheaper than West US).

Create a new Storage Account. The defaults are fine; StorageV2 account type and RA-GRS replication.
Create a new File Share in the Storage Account. In this example, I'll be using a share name of 'templates' which is set to 'Hot'.
Upload the XML file from the previous step to the new file share.
In the Storage Account, go to Access keys and copy the `key1` Connection string. We'll need this for the Azure Automation account.

## Create Azure Automation account

Create a new Azure Automation account in the same Resource Group. When creating it, make sure to leave the option Create Azure Run As account to Yes.

> You can delete the tutorial Runbooks, if you wish.

Go to the Automation account and select Variables under Shared Resources. Here we're going to create a handful of variables for our PowerShell script.

|Variable Name|Value|Encrypted|
|:--- |--- |--- |
|tenantName|tenantName.onmicrosoft.com|No|
|tenantId|GUID_Value|No|
|storageConnString|Connection_string_for_Storage_Account|Yes|
|storageAccount|Name_of_Storage_Account|No|
|fileShare|Name_of_File_Share|No|
|appId|Azure_AD_Registered_AppId|No|

To find the appId and tenantId values, navigate to [Azure AD Registered Applications](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps). You may need to select 'All applications'. The app you're looking for will start with the Azure Automation account name followed by an underscore and random characters. For example, my Azure Automation account name is 'spotemplate' and the registered Azure AD App is named 'spotemplate_4KR84IJsvaNr9HE8yBTri2NKyAdBls7af1FnJ+owyRI='.

Go into the app and copy the 'Application (client) ID' value for the appId variable and 'Directory (tenant) ID' value for the tenantId variable. Keep these values on hand as we'll need them for the Logic App setup, as well.

## Add Client Secret and Graph API Permissions

As we're currently in the Azure AD Registered Application, let's configure the required settings.

We're going to create a Client Secret under 'Certificates & secrets'. Click New client secret. Give it a description and a reasonable expiration time frame (or none at all). I will give mine a description of 'Logic Apps' as that is what we'll use this value for. Copy the new secret as it will not be visible after you navigate away from the page. We'll use the Client Secret later in our Logic App setup.

Next, go to 'API permissions'. Click on 'Add a permission' and scroll down to 'SharePoint'. Click on 'Application permissions' and select 'Sites.FullControl.All'. Next, add the Graph API permission 'Groups.ReadWrite.All'. You can find this under 'Microsoft Graph' and then 'Application permissions'. Note these highly privileged permission in SharePoint Online and for Microsoft 365 Groups, so keep your secrets secret! After adding the permission, make sure to click 'Grant admin consent'.

## Import Modules

Going back to the Azure Automation account you created, go to the 'Modules gallery'. Search for and import the following two modules:

* SharePointPnPPowerShellOnline
* Microsoft.Graph.Authentication

## Create a Runbook

In the Automation account, under 'Runbooks', create a new PowerShell Runbook. In this example, I will create one named 'teamprovisioning'. When created, it will take you to the editor. Paste the code below. Edit the `$fileName` variable on line 15 to indicate the PnP provisioning XML filename you've uploaded to the Azure Storage account. Click Publish.

```powershell
Param
(
    [parameter(Mandatory=$true)]
    [string] $webUrl,
    [parameter(Mandatory=$true)]
    [string] $groupId
)

$appId = Get-AutomationVariable -Name 'appId'
$storageConnectionString = Get-AutomationVariable -Name 'storageConnString'
$fileShareName = Get-AutomationVariable -Name 'fileShare'
$tenantName = Get-AutomationVariable -Name 'tenantName'
$tenantId = Get-AutomationVariable -Name 'tenantId'
$certificate = Get-AutomationCertificate -Name 'AzureRunAsCertificate'
$fileName = 'template.xml'

try {
    Connect-MgGraph -ClientId $appId -TenantId $tenantId -Certificate $certificate
    
    $url = "https://graph.microsoft.com/v1.0/groups/$($groupId)?`$select=hideFromAddressLists"
    $body = @{hideFromAddressLists = 'true'}
    $ApiBody = ConvertTo-Json -InputObject $body
    Invoke-MgGraphRequest -Method PATCH -Uri $url -ContentType 'application/json' -Body $body
    
    $tempFile = New-TemporaryFile
    $storageContext = New-AzureStorageContext -ConnectionString $storageConnectionString
    Get-AzureStorageFileContent -Context $storageContext -ShareName $fileShareName -Path $fileName -Destination $tempFile -Force
    $connection = Connect-PnPOnline -Url $webUrl -Tenant $tenantName -ClientId $appId -Certificate $certificate
    Apply-PnPProvisioningTemplate -Path $tempFile.FullName -Connection $connection
}
catch {
    Write-Output "Error occurred: $PSItem"
}
finally {
    Disconnect-MgGraph
    Disconnect-PnPOnline
}
```

This PowerShell script does two things:

* Connects to the Microsoft Graph and hides the Microsoft 365 Group from the Exchange Online Global Address List
* Applies the PnP template stored in Azure Blob Storage to the Team site

> Communication sites do not have Microsoft 365 Groups so this portion of the script doesn't apply to Communication sites

## Create a Logic App

Logic Apps are the Azure-ified version of Power Automate flows and billed on a consumption basis (there is an option to tie into Azure App Services with always-on, but that's not necessary for this task). Create a new Logic App in your Resource Group.

Go to the Logic App and select 'Logic app designer'. We'll be using the trigger 'When a HTTP request is received'. Start with a Blank Logic App or that trigger.

Add the following actions in this order:

* Parse JSON
* Compose
* Create job (Azure Automation)

In the 'When a HTTP request is received' trigger, add the following JSON schema.

```json
{
    "type": "object",
    "properties": {
      "webUrl": {
        "type": "string"
      },
      "parameters": {
        "type": "object",
        "properties": {
          "event": {
            "type": "string"
          },
          "product": {
            "type": "string"
          }
        }
      }
    }
  }
```

For the 'Parse JSON' action, enter the following JSON schema:

```json
{
    "properties": {
        "createdTimeUTC": {
            "type": "string"
        },
        "creatorEmail": {
            "type": "string"
        },
        "creatorName": {
            "type": "string"
        },
        "groupId": {
            "type": "string"
        },
        "webDescription": {
            "type": "string"
        },
        "webTitle": {
            "type": "string"
        },
        "webUrl": {
            "type": "string"
        }
    },
    "type": "object"
}
```

In the 'Compose' action, add the dynamic content from the Parse JSON action named 'groupId'.

The 'Create job' action will prompt you to sign in, but we're going to 'Connect with Service Principal'. Give the connection a name such as 'azureautomation'. You will need the following from your Azure AD Registered application to connect:

* Tenant ID
* Client ID
* Client Secret

Reminder that the Tenant ID maps to 'Directory (tenant) ID' and the Client ID maps to 'Application (client) ID' values in the registered Azure AD App we previously configured. The Client Secret is the secret value we created in for the registered app.

In the 'Create job' action, click 'Add new parameter' and select 'Runbook Name'. Select the Runbook we previously created. This will then show two new parameters:

* Runbook Parameter groupId
* Runbook Parameter webUrl

If you do not see these values after selecting your Runbook, delete the Create job action and re-create it. You will not need to sign in again. I've run into this bug a couple of times.

Populate the 'groupId' parameter with the dynamic content from the 'Compose' action and the webUrl parameter from the 'When a HTTP request is received' action with the webUrl value.

It should looks similar to the below image.

![pnpprovisioning1.png](/assets/images/2020/12/pnpprovisioning1.png)

Click on the 'When a HTTP request is received' trigger and copy the HTTP POST URL value. We will need this for our site script.

This completes the Azure portion of the configuration. We're now going to jump back to SharePoint and the SPO cmdlets.

## Site Script and Site Design
We'll need to create our site script. Use your favorite JSON editor or just save the following in notepad as a .json file. Set the 'url' value to the HTTP POST URL value you copied from the Azure Logic App in the last step.

```json
{
  "$schema": "schema.json",
  "actions": [
    {
        "verb": "triggerFlow",
        "url": "<HTTP_POST_URL_Value>",
        "name": "TeamSiteDesign"
    }
  ]
}
```

Connect to SharePoint Online using a Global Admin or SharePoint Admin with the SPO PowerShell cmdlets.

```powershell
Connect-SPOService https://contoso-admin.sharepoint.com
$content = Get-Content .\sitescript.json -Raw
$script = Add-SPOSiteScript -Title 'My Custom Team Site' -Content $content
Add-SPOSiteDesign -Title 'My Custom Team Site' -Description 'Team site based on a template' -SiteScripts $script -WebTemplate '64'
```

> Communication sites have a WebTemplate value of '68'.

## Test

The last step in the process to test. Create a Team site from the SharePoint Admin center based on your new site design. Once created, it may take a minute for the PnP template to apply to the site even though the Site Design process has finished. You can check the run of the Logic App and the Azure Automation Runbook to verify they executed successfully.

## Conclusion

Ultimately, this is straight forward to set up and easier than it used to be. I would encourage you to use Azure Logic Apps rather than Power Automate for your flows as Logic Apps aren't tied to an individual (though the trigger and/or actions may be -- but not in this scenario). There's no risk to your provisioning process if an individual leaves the organization.

You can of course use this for Communication sites, like previously mentioned. With Communication sites, there is no Group, so anything referencing/using the 'groupId' variable would need to be modified. In addition, the WebTemplate value is '68' for `Add-SPOSiteDesign`.

Hopefully this was informative and a straight forward guide to implement end-to-end PnP provisioning for Modern sites in SharePoint Online!



