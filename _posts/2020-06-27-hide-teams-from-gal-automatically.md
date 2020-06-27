---
layout: post
title: Hide Modern Team Sites From the Global Address List During Provisioning
tags: [spo]
---

Microsoft [announced](https://developer.microsoft.com/graph/blogs/announcing-support-for-new-groups-properties-via-microsoft-graph-api/) an [addition](https://docs.microsoft.com/graph/api/resources/group?view=graph-rest-beta#properties) to the Graph API beta endpoint for Groups that allows administrators and developers to hide Microsoft 365 Groups from the Exchange Online Address Book at any time, including provisioning of Groups via SharePoint Online Team site creation. This extends to any service that creates a Team site, such as provisioning a Microsoft Team.

This script is designed for Azure Automation, which I would encourage all IT Pros to look at when Site Designs and Power Automate don't cover a particular provisioning scenario. You can attach a Site Design to a site template, including the _default_ Team site template, to call a Power Automate flow which in turn calls Azure Automation to apply any settings you wish.

This does require a registered App in Azure AD with the `Group.ReadWrite.All` API permission. The script has a couple of variables that need to be created in the Azure Automation account Variables section. The script input requires the Power Automate flow to send the Group ID as a variable.

I will have a step-by-step guide for this coming in the near future that covers this script.

Here's the script you would use, in Azure Automation, to hide the Microsoft 365 Groups from the Global Address List in Exchange Online.

```powershell
Param
(
  [Parameter (Mandatory=$true)]
  [String] $groupId
)

$connectionName = "AzureRunAsConnection"

#Get the connection "AzureRunAsConnection"
$servicePrincipalConnection = Get-AutomationConnection -Name $connectionName

try
{
    "Logging in to Azure..."
    Add-AzureRmAccount `
        -ServicePrincipal `
        -TenantId $servicePrincipalConnection.TenantId `
        -ApplicationId $servicePrincipalConnection.ApplicationId `
        -CertificateThumbprint $servicePrincipalConnection.CertificateThumbprint 
}
catch {
    if (!$servicePrincipalConnection)
    {
        $ErrorMessage = "Connection $connectionName not found."
        throw $ErrorMessage
    } else{
        Write-Error -Message $_.Exception
        throw $_.Exception
    }
}

try {
    ### Microsoft 365 Group Hide from GAL
    $ClientSecret = Get-AutomationVariable -Name 'ClientSecret' #from Azure AD registered App
    $loginURL = "https://login.microsoftonline.com"
    $tenantName = Get-AutomationVariable -Name 'TenantName' #contoso.onmicrosoft.com
    $scope = "https://graph.microsoft.com/.default"

    $body = @{grant_type="client_credentials";scope=$scope;client_id=$servicePrincipalConnection.ApplicationId;client_secret=$ClientSecret}
    $oauth = Invoke-RestMethod -Method Post -Uri $("$loginURL/$tenantName/oauth2/v2.0/token") -Body $body
    $token = $oauth.access_token

    # Note we're using a _beta_ endpoint! Update to 1.0 when this API is updated.
    $Uri = "https://graph.microsoft.com/beta/groups/$($groupId)?`$select=hideFromOutlookClients"
    $Header = @{"Authorization" = "Bearer $token"}
    $Body = @{hideFromOutlookClients = 'true'}
    $ApiBody = ConvertTo-Json -InputObject $Body

    Invoke-RestMethod -Uri $Uri -Method PATCH -Headers $Header -Body $ApiBody -ContentType "application/json"
}
catch {
    Write-Output "Error occurred: $PSItem"
}
```

I've waited for years for this property, so I'm glad Microsoft has finally released this capability. It will make your Exchange Online administrators very happy!
