---
layout: post
title: Enabling RBS on multiple content databases
tags: [sp2010, sql]
---

Remote Blog Storage in SharePoint 2010 can be enabled on multiple content databases. The steps you'll need to take are:

1. Run SQL script on first content database
2. Run RBS.msi installation on primary WFE
3. Run RBS.msi installation on secondary WFE, if any
4. Run PowerShell script to enable RBS on content database
5. Run SQL script on second content database
6. Run RBS.msi installation on primary WFE, with different parameters than step 2
7. Run RBS.msi installation on secondary WFE, if any, with different parameters than step 3
8. Run PowerShell script to enable RBS on content database

First, the script:

```sql
USE [ContentDbName]
if not exists (select * from sys.symmetric_keys where name = N'##MS_DatabaseMasterKey##')
create master key encryption by password = N'Admin Key Password !2#4'
USE [ContentDbName]
if not exists (select groupname from sysfilegroups where groupname=N'RBSFilestreamProvider')
alter database [ContentDbName] add filegroup RBSFilestreamProvider contains filestream
USE [ContentDbName]
alter database [ContentDbName] add file (name = RBSFilestreamFile, filename = 'c:\BlobStorage_ContentDbName') to filegroup RBSFilestreamProvider
```

Note that the “Admin Key Password !2#4” value is something you may modify. Also note that the Blob Storage location (`C:\BlobStorage_ContentDbName`) must be unique for each content database.

Once the above script has been run on the first content database, download RBS.msi from the SQL 2008 R2 Feature Pack (download the one listed as RBS_X64.msi).

On your primary WFE, run the following command only for the first content database you are enabling with RBS.

```text
msiexec /qn /lvx* rbs_install_log_ContentDbName.txt /i RBS.msi TRUSTSERVERCERTIFICATE=true FILEGROUP=PRIMARY DBNAME=ContentDbName DBINSTANCE=SQLSERVERNAME FILESTREAMGROUP=RBSFilestreamProvider
```

Obviously change the “ContentDbName” parameters to the content database you ran the SQL script against. The DBINSTANCE can either be localhost (if SQL is installed locally), the remote SQL Server name, or SQL Server Name and instance name (e.g. SQLSERVERINSTANCE). The FILESTREAMGROUP parameter should be identical to the “filegroup” in the SQL Script on line 8 above.
Next, on the secondary WFE, if you have one, again you will need RBS.msi to run the following:

```text
msiexec /qn /lvx* rbs_install_log_ContentDbName.txt /i RBS.msi DBNAME=ContentDbName DBINSTANCE=SQLSERVERNAME ADDLOCAL="Client,Docs,Maintainer,ServerScript,FilestreamClient,FilestreamServer"
```
Validate that the RBS tables exist in the Content Database by running the following SQL script on the Content Database:

```sql
USE [ContentDbName]
select * from dbo.sysobjects
where name like 'rbs%'
```

If tables are returned, the RBS provider has been installed successfully. If no tables are returned, check the RBS install log and also check for typos in your msiexec command.
Now, on either WFEs, run the following PowerShell script using the SharePoint 2010 Management Shell:

```powershell
$cdb = Get-SPContentDatabase ContentDbName
$rbs = $cdb.RemoteBlobStorageSettings
$rbs.Installed() #This should return a result of True
$rbs.Enable()
$rbs.SetActiveProviderName($rbs.GetProviderNames()[0])
```

This should initialize RBS for the first content database. To test, upload a file to a SharePoint site covered by the Content Database that is over 100KB, then check the Blob Storage location as defined by the SQL script. A new folder with a GUID should be created, along with a sub folder  and there should be at least one file present in the directory which should be the size of your uploaded file. Another way to check if RBS is functional is to run:

```powershell
$rbs.Migrate()
```

This will migrate all eligible files out of the content database into the RBS file stream. Note this operation may take some time to complete depending on the size of the content in the database.

Another option you may want to set is to limit the files eligible to participate in RBS. To do this, run:

```powershell
$rbs.MinimumBlobStorageSize 524288 #value in bytes
```

For the second, or more content databases, again run the SQL script, changing the values of ContentDbName where present to reflect the Content Database.

Instead of the original msiexec command, run the following msiexec command on subsequent databases:

```text
msiexec /qn /lvx* rbs_install_log_ContentDbName.txt /i RBS.msi REMOTEBLOBENABLE=1 FILESTREAMPROVIDERENABLE=1 DBNAME=ContentDbName ADDLOCAL="EnableRBS,FilestreamRunScript" DBINSTANCE=SQLSERVERNAME
```

On the secondary WFE, run:

```text
msiexec /qn /lvx* rbs_install_log_ContentDbName.log /i RBS.msi DBNAME=ContentDbName DBINSTANCE=SQLSERVERNAME ADDLOCAL="EnableRBS,FilestreamRunScript"
```

After the script runs successfully, run the PowerShell script against the second content database. Again, uploading a file should work, as should $rbs.Migrate() if you choose to do that.

Make sure to look at the information outlined in the TechNet article Maintain Remote BLOB Storage (RBS) for ongoing maintenance of RBS-enabled Content Databases.

4/3/2011 EDIT: Some have said that they've been unable to get this working properly.  Note that this particular SharePoint instance, pictured below, is a single server (Complete) installation (WFE/App/SQL on one VM).  The SQL script should not be a mystery as it doesn't change from database-to-database (besides database name and RBS file path).  Ignore my errors :)

Installing the RBS provider on the first and second database.  One thing to note is that I have waited between the installations for the RBS provider installation to finish, by tracking the MsiInstaller Application event log entries.  The first RBS.msi installation should yield an MsiEvent 1033:

```text
Log Name: Application
Source: MsiInstaller
Date: 4/3/2011 12:55:01 PM
Event ID: 1033
Task Category: None
Level: Information
Keywords: Classic
User: NAUPLIUSs-setup
Computer: SP01.nauplius.local
Description:
Windows Installer installed the product. Product Name: SQL Server 2008 R2 Remote Blob Store. Product Version: 10.50.1600.1. Product Language: 1033. Manufacturer: Microsoft Corporation. Installation success or error status: 0.
```

The second (and subsequent) RBS.msi installations should yield an MsiEvent 1035:

```text
Log Name:      Application
Source:        MsiInstaller
Date:          4/3/2011 12:58:10 PM
Event ID:      1035
Task Category: None
Level:         Information
Keywords:      Classic
User:          NAUPLIUSs-setup
Computer:      SP01.nauplius.local
Description:
Windows Installer reconfigured the product. Product Name: SQL Server 2008 R2 Remote Blob Store. Product Version: 10.50.1600.1. Product Language: 1033. Manufacturer: Microsoft Corporation. Reconfiguration success or error status: 0.
```

And here is an image of the command prompt.

![RBSInstall](/assets/images/2011/03/RBSInstall.png)

And then the entire process for the SharePoint Management Shell, from confirming the RBS installation to migrating content from both content databases via the RBS provider.

![RBSEnableAndMigrate](/assets/images/2011/03/RBSEnableAndMigrate.png)

![RBSEnableAndMigrate2](/assets/images/2011/03/RBSEnableAndMigrate2.png)


And here is the file system containing two files each (all that was uploaded to WebApp1 and WebApp2) these files can be opened by Word (they're both docx) or Notepad/Wordpad if Word is not installed, to view their content.

![RBSFileSystem1](/assets/images/2011/03/RBSFileSystem1.png)

![RBSFileSystem2](/assets/images/2011/03/RBSFileSystem2.png)

Hopefully this helps anyone struggling to get RBS enabled on multiple content databases.  If anyone has any questions, please let me know.  I may also be able to make a movie of the entire process for two content databases, if that could be helpful.