---
layout: post
title: Creating a SQL Server Availability Group via PowerShell
tags: [sp2010,sp2013,sql]
---

This is tangibly related to SharePoint and this took me awhile to figure out the correct syntax.  I've been building a SQL AG on Server 2012 with the Core installation option.  To accomplish this, you will need a workstation with SQL Management Studio installed (which also installed the SQL PowerShell module).  Some background on this setup:

2 servers with Windows Server 2012 and SQL Server 2012 SP1 CU5

3 network adapters per server

Features installed (note that "D:" has the Windows Server ISO loaded) on the servers:

```powershell
Install-WindowsFeature Failover-Clustering,NET-Framework-Core -Source:D:\sources\sxs
```

IP all of the adapters appropriately.  Next, rename the secondary and tertiary network adapters for clarity:

```powershell
Rename-NetAdapter "Ethernet 2" -NewName "Data"
Rename-NetAdapter "Ethernet 3" -newName "Heartbeat"
```

Note that you may want to disable or modify the Firewall Policy for the "Domain" and "Private" profiles.  You can do this via the Set-NetFirewallProfile cmdlet.

On the client workstation with the RSAT tools installed (Failover Cluster), run the following to import the Failover Cluster module, create the cluster with a static IP address, then rename the network adapters to fit the adapter name on the server.  Finally, set the Ethernet adapter (this is the adapter that is used for standard client to server communication) Host Record TTL to 300 seconds.

```powershell
Import-Module FailoverClusters
New-Cluster -Name "SQLCLUSTER" -Node "SQLSERVER1","SQLSERVER2" -StaticAddress "192.168.0.20"
$n1 = Get-ClusterNetwork -Cluster "SQLCLUSTER" -Name "Cluster Network 1"
$n1.Name = "Heartbeat"
$n2 = Get-ClusterNetwork -Cluster "SQLCLUSTER" -Name "Cluster Network 2"
$n2.Name = "Data"
$n3 = Get-ClusterNetwork -Cluster "SQLCLUSTER" -Name "Cluster Network 3"
$n3.Name = "Ethernet"

Get-ClusterResource "Ethernet" | Set-ClusterParameter -ClusterParameter HostRecordTTL 300
```

Next, on each server, install SQL via batch script.  Real quick hint here, make sure to exit the PowerShell prompt prior to running this.  The SQL installation media needs .NET 3.5, but because we're running Server 2012 and PowerShell 3.0, we're using .NET 4.  This will cause the installation to fail.  The following will install SQL Server 2012 with the Database Engine, SQL Agent, Replication, Integration Services, and Client Connectivity with the specified username and password to the E: drive while the CD/ISO is present in the D: drive.  Do not forget to edit the /PID value.  It will also enable TCP/IP connectivity to the instance.

```powershell
@ECHO OFF
set Domain=DomainName
set SQLUser=ServiceAccount
set SQLPass=Password
set SQLRoot=E:
set CDRoot=D:
@ECHO ON

msiexec.exe /i %CDROOT%\1033_ENU_LP\x64\Setup\sqlsupport_msi\SqlSupport.msi /passive /norestart

%CDROOT%\Setup.exe /Q /ACTION="INSTALL" /IACCEPTSQLSERVERLICENSETERMS /UpdateEnabled=0 /ERRORREPORTING=0 /INSTANCENAME=MSSQLSERVER /FEATURES=SQLEngine,Replication,IS,Conn,ADV_SSMS /INSTANCEDIR="%SQLRoot%\Program Files\Microsoft SQL Server" /AGTSVCACCOUNT="%Domain%\%SQLUser%" /AGTSVCPASSWORD="%SQLPass%" /AGTSVCSTARTUPTYPE="Automatic" /INSTALLSQLDATADIR="%SQLRoot%\Program Files\Microsoft SQL Server" /SQLSVCACCOUNT="%Domain%\%SQLUser%" /SQLSVCPASSWORD="%SQLPass%" /SQLSVCSTARTUPTYPE="Automatic" /SQLSYSADMINACCOUNTS="%Domain%\Domain Admins" /SQLTEMPDBDIR="%SQLRoot%\Program Files\Microsoft SQL Server\MSSQL11.MSSQLSERVER\MSSQL\DATA" /SQLTEMPDBLOGDIR="%SQLRoot%\Program Files\Microsoft SQL Server\MSSQL11.MSSQLSERVER\MSSQL\DATA" /SQLUSERDBDIR="%SQLRoot%\Program Files\Microsoft SQL Server\MSSQL11.MSSQLSERVER\MSSQL\DATA" /SQLUSERDBLOGDIR="%SQLRoot%\Program Files\Microsoft SQL Server\MSSQL11.MSSQLSERVER\MSSQL\DATA" /ISSVCACCOUNT="%Domain%\%SQLUser%" /ISSVCPASSWORD="%SQLPass%" /ISSVCStartupType="Automatic" /PID="XXXXX-XXXXX-XXXXX-XXXXX-XXXXX" /INDICATEPROGRESS=1 /TCPENABLED=1
```

Once complete, download [SQL Server 2012 SP1](http://www.microsoft.com/en-us/download/details.aspx?id=35575) and I've chosen to also install [CU5](http://support.microsoft.com/kb/2861107).  Once downloaded, extract the executable using the following command, making sure to extract each package to a unique path:

`<executable file name>.exe /extract:C:\sp1`

Next, for each package, run:

```powershell
setup.exe /qs /IAcceptSQLServerLicenseTerms /Action=Patch /AllInstances
```

This will install the package applying the patch to all instances on the server.  You may or may not need to reboot in between patches.

Next, move to the workstation.  I have a workstation running Windows 8 x64 with SQL Management Studio 2012 SP1.  Open Management Studio and go to View -> Registered Servers.  Add the two instances under Local Server Groups.

Next, run PowerShell as Administrator on the Workstation.  Run `sqlps` or `Import-Module sqlps` to import the SQL PowerShell module.  Validate the previous server registration by executing `cd SQLSERVER:\SQLRegistration\"Database Engine Server Group"` , and running `ls`.  Both servers should appear here.  The next step is to connect to each server, which can be done by executing cd `SQLSERVER:\SQL\SERVERNAME`.

For each SQL Server, the next step is to enable AlwaysOn and create the HADR Endpoint.  Both cmdlets have a few options, so review them prior to execution.  Note that when enabling AlwaysOn, the Database Engine service must be restarted, which the -Force switch does (or should do, it didn't work in my case).

```powershell
Enable-SqlAlwaysOn -Force
New-SqlHADREndPoint -Name "SPHADR" -Encryption 'Required'
```

Next, create the Availability Replicas (in memory), create the Availability Group with the Primary server specified, and finally Join the secondary server to the Availability Group.  Again, these cmdlets have a lot of options, so it is best to review them so the setup fits your environment.

```powershell
$ps = New-Object Microsoft.SQLServer.Management.SMO.Server("SQLSERVER1")
$ss = New-Object Microsoft.SQLServer.Management.SMO.Server("SQLSERVER2")

$ps = (gi SQLSERVER:\SQL\SQLSERVER1\DEFAULT)
$ss = (gi SQLSERVER:\SQL\SQLSERVER2\DEFAULT)
$pr = New-SqlAvailabilityReplica -Name "SQLSERVER1" -EndpointUrl "TCP://SQLSERVER1.domain.com:5022" -FailoverMode "Automatic" -AvailabilityMode "SynchronousCommit" -BackupPriority 50 -ConnectionModeInSecondaryRole AllowAllConnections -ReadonlyRoutingConnectionUrl TCP://SQLSERVER1.domain.com:1433  -Version ($ps.Version) -AsTemplate

$sr = New-SqlAvailabilityReplica -Name "SQLSERVER2" -EndpointUrl "TCP://SQLSERVER2.domain.com:5022" -FailoverMode "Automatic" -AvailabilityMode "SynchronousCommit" -BackupPriority 0 -ConnectionModeInSecondaryRole AllowAllConnections -ReadonlyRoutingConnectionUrl TCP://SQLSERVER2.domain.com:1433 -Version ($ss.Version) -AsTemplate

New-SqlAvailabilityGroup -InputObject $ps -Name SharePointAG -AvailabilityReplica ($pr, $sr)
Join-SqlAvailabilityGroup -InputObject $ss -Name SharePointAG
```

After this, via Management Studio, you should now be able to review the Availability Group status.  In my case, I had a critical error which was due to my HADR endpoint being in a stopped state, preventing the secondary replica from connecting to the primary.  To resolve this, I ran the following T-SQL:

```sql
ALTER STATE [SPHADR] SET STATE = STARTED;
```

Once this completed, the secondary replica joined automatically.  Note it is normal to have warnings as there are no synchronized databases at this point.

The final step to create the Availability Group is to create the Listener, which can be done with the following cmdlet:

```powershell
New-SqlAvailabilityGroupListener -Name SPAGL -StaticIp '192.168.0.21/255.255.255.0' -Path SQLSERVER:\SQL\SQLSERVER1\DEFAULT\AvailabilityGroups\SharePointAG
```

And now you have two complete SQL Servers, ready to have SharePoint databases added to the Aavailabity Group!  Make sure to test failover to validate functionality.  Do not forget the following resources with regards to supportability of SharePoint databases on an Availability Group.

* [Supported high availability and disaster recovery options for SharePoint databases (SharePoint 2013)](http://technet.microsoft.com/en-us/library/jj841106.aspx)

* [Configure SQL Server 2012 AlwaysOn Availability Groups for SharePoint 2013](http://technet.microsoft.com/en-us/library/jj715261.aspx)