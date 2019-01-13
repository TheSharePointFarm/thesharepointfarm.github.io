---
layout: post
title: SharePoint Database Availability Group Cmdlets
tags: [sp2013]
---

The SharePoint 2013 April 2014 Cumulative Update includes three new SQL Availability Group cmdlets:

* `Get-AvailabilityGroupStatus`
* `Add-DatabaseToAvailabilityGroup`
* `Remove-DatabaseFromAvailabilityGroup`

These cmdlets allow you to manage the SQL Availability Group. First, a few notes about the Availability Group.

Note, the below information about the AG needing to reside on the SQL instance with the Configuration database has changed in later releases. You may use other Availability Groups that do not contain the SQL instance with the Configuration database. This is only true when not using the `-ProcessAllDatabases` switch, of course.

The AG must be the connection in use by the Configuration database as checks for the Database Availability group will be executed against this connection. If the Database Availability group happens to be on another set of SQL Servers that is not the Configuration database SQL server connection, those checks will fail. The Availability Group must have a Database Availability Group Listener. Again, without it, the checks will fail.

To confirm that the check will succeed, modify the AGName parameter in this T-SQL query and run it against the SQL instance used by the Configuration database:

```sql
SELECT dns_name, port FROM sys.availability_group_listeners l 
JOIN sys.availability_groups g ON g.group_id = l.group_id
WHERE name = 'AGName'
```

Let's start with a brand new farm. Two SQL Servers, SQLAO1 (Primary), and SQLAO2 (Secondary) in a SQL Availability Group (Synchronous mode) named SPAG (DNS name is SPAG.nauplius.local). A dummy test database has been created in order to set up the AG, but may be deleted. The SharePoint server is SharePoint 2013 SP1 with April 2014 CU. No SQL Alias will be used for this set up since we have database mobility between SQL Servers due to AlwaysOn. Starting with an elevated SharePoint Management Shell...

```powershell
New-SPConfigurationDatabase -DatabaseName "Configuration" -DatabaseServer "SPAG" -AdministrationContentDatabaseName "Administration" -FarmCredentials (Get-Credential) -Passphrase (ConvertTo-SEcureString "PassPhrase1" -AsPlainText -Force)
Install-SPHelpCollection -All
Initialize-SPResourceSecurity
Install-SPService -AllExistingFeatures
Install-SPApplicationContent
New-SPCentralAdministration -Port 8080 -WindowsAuthProvider "NTLM"
```

Because SQLAO1 is the Primary Replica, the Administration and Configuration databases are present on it, but not SQLAO2. The next step is where the AlwaysOn fun begins. Validate a good backup location is available for each SQL Server (note that your SQL Server service account(s) will need access to this location). Run the following command from the SharePoint Management Shell:

```powershell
Add-DatabaseToAvailabilityGroup -AGName "SPAG" -FileShare "\\SQLAO1\Backup" -ProcessAllDatabases
```

And that is it! Your Configuration and Administration databases are now part of the SQL Availability Group (in a real sense), there was no having to flip-flop SQL Aliases around, the databases are automatically backed up and synchronized, as are the SQL Logins! This also eliminates the need for a Database Administrator to assist with getting SharePoint databases into an Availability Group, because it can now be done by the SharePoint Administrator!

You can also now see the status of the Availability Group:

```powershell
PS C:\Users\trevor> Get-AvailabilityGroupStatus
Listener                    : SPAG
Replicas                    : {SQLAO1, SQLAO2}
Name                        : SPAG
TypeName                    : Microsoft.SharePoint.Administration.SPAvailabilityGroup
DisplayName                 : SPAG
Id                          : b803800b-790c-43ab-9ad1-6728b914e0bf
Status                      : Online
Parent                      : SPFarm Name=Configuration
Version                     : 8538
Properties                  : {}
Farm                        : SPFarm Name=Configuration
UpgradedPersistedProperties : {}
```

If you run `Add-DatabaseToAvailabilityGroup`, but the cmdlet errors out, an Availability Group object may be created within SharePoint regardless. To fix this, simply run:

```powershell
$ag = Get-AvailabilityGroupStatus | where {$_.Name -eq "AGName"}
$ag.Delete()
```
For databases that are part of the SharePoint `SPAvailabilityGroup` (rather than simply being in an AG as done from SQL Management Studio) will also be aware of that through their object.

```powershell
$config = Get-SPDatabase | where {$_.Name -eq "Configuration"}
$config.AvailabilityGroup

Listener                    : SPAG
Replicas                    : {SQLAO1, SQLAO2}
Name                        : SPAG
TypeName                    : Microsoft.SharePoint.Administration.SPAvailabilityGroup
DisplayName                 : SPAG
Id                          : b803800b-790c-43ab-9ad1-6728b914e0bf
Status                      : Online
Parent                      : SPFarm Name=Configuration
Version                     : 8538
Properties                  : {}
Farm                        : SPFarm Name=Configuration
UpgradedPersistedProperties : {}
```

On the SharePoint `SPAvailabilityGroup` object itself, we can also force a failover to one of other nodes in the Availability Group:

```powershell
$ag = Get-AvailabilityGroupStatus
$ag.ForceFailover("SQLAO2","SQLAO1")
```

Yep, as a SharePoint Administrator, I do not have to touch SSMS or SQL PowerShell to execute an AG failover anymore!

If, down the road, you create a new database (Content or Service Application) and need to individually add it to a new or existing Availability Group, again use the `Add-DatabaseToAvailabilityGroup` cmdlet. Pay attention to what the Primary replica is, as the last backup path will be used if the `-FileShare` switch is not specified. So for my above example where my FileShare path is on SQLAO1, but I've failed over to SQLAO2, if I attempt to use the Add cmdlet, the backup will fail with Access Denied. Instead, I created a new share on SQLAO2 and ran:

```powershell
Add-DatabaseToAvailabilityGroup -AGName "SPAG" -DatabaseName "SPAGFOTST" -FileShare "\\SQLAO2\Backup"
```

Of course, you could use a common CIFS location that the SQL Server service accounts had NTFS Modify access to, as well.

The last cmdlet, `Remove-DatabaseFromAvailabilityGroup`, will remove databases from the SharePoint `SPAvailabilityGroup` object. If you attempt to run the cmdlet against a database that currently exists within SharePoint (for example, is still attached to a Web Application), the cmdlet will fail with an error similar to:

```powershell
Remove-DatabaseFromAvailabilityGroup : Operation cannot be performed because
|0 still exists in SharePoint. To overwrite, specify the -Force parameter.
```

This is a little bit misleading (and there is a slightly better error in the ULS logs). If you use the `-Force` switch, what it does is it removes it from the Availability Group, deleting the database from the Secondary nodes of the SQL Availability Group (by default), but the database will still be attached to SharePoint. If you need to remove a database from an Availability Group, but wish to keep copies of the database on the Secondary nodes, use the `-KeepSecondaryData` parameter. The database, on the Secondary nodes, will enter a Not Synchronizing state, while the database with the active connection will no longer display a synchronizing state as it is no longer part of the Availability Group.

One potential bug to note is that it appears the Secondary replicas do not have the Max Degree of Parallelism (MAXDOP) set to 1 at any point by SharePoint. Make sure to set it manually prior to deploying SharePoint databases as this can cause certain operations to fail, such as creating new Content Databases.

The December 2014 CU has added this note when adding the Configuration database to the Availability Group via the `Add-DatabaseToAvailabilityGroup` cmdlet:

```powershell
Adding configuration database. You must restart all SharePoint servers to complete this action.
```