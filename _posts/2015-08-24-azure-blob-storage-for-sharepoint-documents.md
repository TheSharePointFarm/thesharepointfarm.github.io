---
layout: post
title: Azure Blob Storage for SharePoint Documents
tags: [sp2016]
---

[This post is based on the SharePoint 2016 Preview — Functionality is not guaranteed to be supported, configured identically, or exist in RTM. This functionality is highly likely to not exist in the RTM product and is shown here as an academic exercise.]

SharePoint 2016 includes some very interesting features that allow us to take a look at what is being used within SharePoint Online ("the service"). One of these features is Fort Knox, which is per-file encryption blobs residing in Azure, instead of SharePoint's Content Database. Leveraging Azure Blob Storage for SharePoint documents is not a straight forward process, and is currently not supported by the SharePoint Product Group, but below are the steps to follow in order to enable it for demonstration purposes. If support for this is interesting for you or your customers, make sure to let the SharePoint Product Group know via [UserVoice](http://sharepoint.uservoice.com/) or an RFC through standard Microsoft support offerings. If this process was ever supported, the assumption would be that it will only be supported for SharePoint farms running in Azure Virtual Machines and where the farm is in the same Azure region as the primary Azure Blob Storage account.

To set this up, you'll need to have an Azure Subscription, and an Azure Active Directory user account (this user account does not need any assigned license within Office 365, if your Azure subscription is tied to Office 365).

Create two Azure Blob Storage accounts. These accounts have the following requirements:

1) Must start with the letters "spo".

2) Must end with the letter "c".

3) Cannot be longer than 10 characters total, including "spo", and "c". In my examples, I used spo1splabc and spo2splabc.

4) Must have two Azure Blob Storage accounts. This will not work without two accounts!

One Storage will be used as the Primary, the other as a Secondary (or "DR"). Give the Azure AD account Owner rights over the Azure Blob Storage accounts. The container may stay Private R/W.

On your SharePoint server(s), install [Microsoft.WindowsAzure.Storage.dll](https://www.nuget.org/packages/WindowsAzure.Storage/3.0.3). You can use `GACUtil` to install the binary to the GAC. The last pre-req is to set the SharePoint Farm Credentials to connect to Azure.

```powershell
$farm = Get-SPFarm
$farm.SetFarmLevelAzureCredentials("azureUser@azureaddomain.com", (ConvertTo-SecureString "azureUserPassword" -AsPlainText -Force), "SPServerAdmin", $true, $true, $true, $true)
```

You must create Shared Access Signature (SAS) URIs. Using the [GenerateSas](https://azure.microsoft.com/en-us/documentation/articles/storage-dotnet-shared-access-signature-part-2/) project, you will modify the app.config included with the project with the Connection String from your Azure Blob Storage account. This key will look like:

```
DefaultEndpointsProtocol=https;AccountName=spo1splabc;AccountKey=<key>
```

You must generate a unique SAS URI per Blob Storage account. Note that if you want to change the container name used (as this will be used elsewhere during this guide), adjust this line, changing "sascontainer" to your desired value. The container value must be a minimum of 3 characters. This guide will continue using "sascontainer" as the value, since it is the default included with the project.

```csharp
    //Get a reference to a container to use for the sample code, and create it if it does not exist.
    CloudBlobContainer container = blobClient.GetContainerReference("sascontainer");
```

In addition, there is another change to this project, which sets the expiration value at the maximum and grants further permissions.

```csharp
            //Set the expiry time and permissions for the container.
            //In this case no start time is specified, so the shared access signature becomes valid immediately.
            SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy();
            sasConstraints.SharedAccessExpiryTime = DateTimeOffset.MaxValue;
            sasConstraints.Permissions = SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.List | SharedAccessBlobPermissions.Delete | SharedAccessBlobPermissions.Read;
```

Take the output of the console application and place it into a PowerShell variable, one for each container. For example:

```powershell
$fullUri = "https://spo1splabc.blob.core.windows.net/sascontainer?sv=2013-08-15&sr=c&sig=<signature>&se=9999-12-31T23%3A59%3A59Z&sp=rwdl"
$fullUriFar = "https://spo2splabc.blob.core.windows.net/sascontainer?sv=2013-08-15&sr=c&sig=<signature>&se=9999-12-31T23%3A59%3A59Z&sp=rwdl"
```

Next, grab the SharePoint Farm Id.

```powershell
$id = (Get-SPFarm).Id
```

This next cmdlet is lengthy. You'll also note I'm not strictly following the purpose of each parameter, instead using the same key for multiple purposes.

```powershell
Update-SPAzureBlobSignaturesEx -ReadSignaturesByPrimaryKey $fullUri -WriteSignaturesByPrimaryKey $fullUri -ReadSignaturesBySecondaryKey $fullUri -WriteSignaturesBySecondaryKey $fullUri -DRReadSignaturesByPrimaryKey $fullUriFar -DRWriteSignaturesByPrimaryKey $fullUriFar -DRReadSignaturesBySecondaryKey $fullUriFar -DRWriteSignaturesBySecondaryKey $fullUriFar -PrimaryFarmPoolId $id.Guid -DRFarmPoolId $id.Guid
```

This adds the keys to the SharePoint server's registry. Provide the WSS_WPG account with Read/Write permissions to the following key, which needs to be done on each SharePoint server.

```
HKLM\SOFTWARE\Microsoft\Shared Tools\Web Server Extensions\16.0\Secure\Credentials
```

Run the following PowerShell cmdlet:

```powershell
$enc = [System.Text.Encoding]::UTF8 
$string = "This is a string" 
$bytes = $enc.GetBytes($string) 
Update-SPAzureBlobConfigLocator -Locator $bytes
```

Create the following file on each server, which needs to have the same content on each server.

```
C:\Program Files\Common Files\microsoft shared\Web Server Extensions\16\CONFIG\AbsBlobStoreConfig.xml
```

The content of this file needs to be, at a minimum, this:

```xml
<?xml version="1.0"?> 
<ArrayOfHashTableItem> 
<HashTableItem/> 
</ArrayOfHashTableItem>
```

The next steps are to modify the database directly via T-SQL. You must set these values correctly for the Azure Blob Storage upload to take place.

```sql
_EnableDirectWriteToAbs = true 
_start_draintoabs_TimerJob = true
```

The first allows for SharePoint to directly write to ABS while the second allows the ABS Drain Timer Job to run (drain meaning "move all content to ABS"). In addition, you must provide where to migrate the blobs to along with where to store the blobs!

```sql
_MigrationTargetAbsLocationInfo = spo1splabc.blob.core.windows.net;/sascontainer;spo2splabc.blob.core.windows.net;/sascontainer;1;9999-12-31 23:59:59
_AbsLocationInfo = spo1splabc.blob.core.windows.net;/sascontainer;spo2splabc.blob.core.windows.net;/sascontainer;1;9999-12-31 23:59:59
```

The format for `_AbsLocationInfo` and `_MigrationTargetAbsLocationInfo` is `NearABSFQDN;/NearContainerName;FarABSFQDN;/FarContainerName;BlobExpirationTime`. To add all of these values to the SharePoint Content Database, use the following T-SQL, where the variable is, for example, "_EnableDirectWriteToAbs" and the value is "true".

```sql
Use [ContentDatabaseName]
INSERT INTO dbo.DatabaseInformation VALUES ('variable', 'value')
```

Finally, you're ready to migrate to Azure Blob Storage! Run the job-drain-docstreams-abs timer job:

```powershell
$job = Get-SPTimerJob job-drain-docstreams-abs
$job.RunNow()
```

This will execute the job, and if everything was set up correctly, encrypt and migrate your data from the DocStreams table to Azure Blob Storage. The result will be similar to this:

![ABSDocStreamsTable](/assets/images/2015/08/ABSDocStreamsTable.png)

The `ABSId` value corresponds to the file name of the Blob, as a GUID while `ABSInfo` is information about the ABS Container, version, and encryption information. Here we can see the corresponding ABSId, 0EB745E7-... in the Azure Blob Storage container:

![ABSBlobStoreDoc](/assets/images/2015/08/ABSBlobStoreDoc.png)

As new documents are uploaded to the Content Database, files will be written to ABS instead of the Content Database.

If for some reason the migration from the Content Database to Azure Blob Storage fails and you want to retry after resolving any issues, you must perform two steps:

```sql
UPDATE the value of _AbsLocationInfo in the database -- it resets back to a default value after the drain job runs.
DELETE the variable _SkipDrainStreamUntilTomorrow from the DatabaseInformation table.
```