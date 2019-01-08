---
layout: post
title: Using Application Request Routing as a Reverse Proxy for SharePoint
tags: [sp2007,sp2010,sp2013]
---

With the questionable life span of the Microsoft Forefront brand, the Application Request Routing module for IIS7+ serves as a replacement reverse caching proxy.  In conjunction with the Web Farm Framework and URL Rewrite, the ARR, in some cases, can provide an alternative to licensed products, such as Microsoft UAG, for todays needs.  This guide will walk you through creating an ARR server running Windows Server 2012 Core to proxy requests to a SharePoint 2010 server (with notes for SharePoint 2013).

To start, build a Windows Server 2012 installation using the Core install option.  The core install option is ideal to reduce the attack surface of our reverse proxy and increase uptime via the way of a reduced patching scope.

Once the server is built, set the Network Adapter configuration via PowerShell:

```powershell
Get-NetAdapter #To identify existing adapter(s)
Rename-NetAdapter -Name "Ethernet" -NewName "Internet"
New-NetIPAddress -AddressFamily IPv4 -IPAddress 10.10.20.28 -PrefixLength 24 -DefaultGateway 10.10.20.1 -InterfaceAlias "Internet"
Set-DnsClientServerAddress -ServerAddresses 10.10.20.8 -InterfaceAlias "Internet"
```

Note that the DNS servers that the network adapter uses must resolve the SharePoint host names back to the SharePoint server(s)!  If the client entry here resolves SharePoint host names back to itself, you may face repeated authentication prompts from the ARR server.

Next, rename the server and restart.  You'll note that we are not joining a domain here.  Joining a domain is optional and may increase the attack surface of the server depending on network configuration.  In addition, joining a domain is not a requirement for creating an ARR (or Windows NLB) farm!

```powershell
Rename-Computer ARR01
Restart-Computer
```

When the server is back online, add the necessary Windows Features:

```powershell
Add-WindowsFeature Web-Static-Content,Web-Mgmt-Service
```

Yep, only two features (well, it adds dependencies...)!

Next, we need to transfer the necessary bits to the server in order to install the Application Request Routing pre-requirements and patches.

```powershell
#URL Rewrite 2:
Start-BitsTransfer -Source http://download.microsoft.com/download/6/7/D/67D80164-7DD0-48AF-86E3-DE7A182D6815/rewrite_amd64_en-US.msi
#Web Farm Framework 1.1:
Start-BitsTransfer -Source http://download.microsoft.com/download/5/7/0/57065640-4665-4980-A2F1-4D5940B577B0/webfarm_v1.1_amd64_en_US.msi
#Application Request Routing 2.5:
Start-BitsTransfer -Source http://download.microsoft.com/download/A/A/E/AAE77C2B-ED2D-4EE1-9AF7-D29E89EA623D/requestRouter_amd64_en-US.msi
#External Disk Cache v1:
Start-BitsTransfer -Source http://download.microsoft.com/download/3/4/1/3415F3F9-5698-44FE-A072-D4AF09728390/ExternalDiskCache_amd64_en-US.msi
#External Disk Cache Patch:
Start-BitsTransfer -Source http://download.microsoft.com/download/D/E/9/DE90D9BD-B61C-43F5-8B80-90FDC0B06144/ExternalDiskCachePatch_amd64.msp
#For IIS8 only, Application Request Routing hotfix:
Start-BitsTransfer -Source http://download.microsoft.com/download/6/2/6/6260674B-A3CA-434D-A538-561087EB5D04/requestrouter_amd64.msp
#Application Request Routing hotfix for NTLM support:
Start-BitsTransfer -Source http://download.microsoft.com/download/5/C/3/5C3FFDA4-7A8F-4065-9B4C-327D87569707/requestrouter_amd64_KB2732764.msp
```

We will install the bits the same order we downloaded them in:

```text
rewrite_amd64_en-US.msi /passive
webfarm_v1.1_amd64_en_US.msi /passive
requestRouter_amd64_en-US.msi /passive
ExternalDiskCache_amd64_en-US.msi /passive
ExternalDiskCachePatch_amd64.msp /passive
requestrouter_amd64.msp /passive #IIS8 only
Requestrouter_amd64_KB2732764.msp
```

Next, you will need to transfer the SharePoint server SSL certificate with private key and certificate chain to the Application Request Routing server.  Here's a hint: You can use Start-BitsTransfer for that, too!

Export the SSL certificate from your SharePoint server with the private key to a PFX file.  Copy the PFX to a location to transfer it from.

```powershell
Start-BitsTransfer -Source https://webserver.company.com/certificate.pfx
```

Next, we'll import the certificate and certificate chain.  If the certificate chain is not imported, we are likely to receive "502 Bad Gateway" errors when attempting to view SharePoint sites via SSL.

```powershell
$certCollection = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2Collection
$certCollection.Import("C:\Users\Administrator\Downloads\Certificate.pfx","SecretPassword",[System.Security.Cryptography.X509Certificates.X509KeyStorageFlags]"PersistKeySet,MachineKeySet")
```

We must add each certificate to the appropriate store.  This PFX only has two certificates in it, the Root Certificate Authority and the Wildcard SSL Certificate.  When viewing the `$certCollection` object, the certificates are ordered in a zero-based index.  We can access individual certificates based on this index.

![CertificateCollection](/assets/images/2013/05/CertificateCollection.png)

Create two X509Store objects to add each certificate to.  This example only requires the "MY" (Personal) and "Root" (Trusted Root Certificate Authorities).  We will be adding them to the LocalMachine store.

```powershell
$certStore1 = New-Object System.Security.Cryptography.X509Certificates.X509Store("Root","LocalMachine")
$certStore2 = New-Object System.Security.Cryptography.X509Certificates.X509Store("My","LocalMachine")

$certStore1.Open("MaxAllowed")
$certStore1.Add($certCol[0])
$certStore1.Close()

$certStore2.Open("MaxAllowed")
$certStore2.Add($certCol[1])
$certStore2.Close()
```

Next, we will want to manage the Application Request Routing server from a remote machine using IIS Manager.  You will either need Windows Server 2012 with the IIS Manager installed, or Windows 8 with the [IIS Manager for Remote Administration](http://www.iis.net/downloads/microsoft/iis-manager) installed.  The built-in IIS Manager included with Windows 8 does not allow for remote administration.  To enable remote management on the Application Request Routing server, run:

```powershell
sp HKLM:\SOFTWARE\Microsoft\WebManagement\Server EnableRemoteManagement -Value 1
Start-Service WMSVC
Set-Service WMSVC -Startup Automatic
```

This will flip the bit on the `EnableRemoteManagement` key, start the IIS Web Management Service, and set the service to automatically start.

Using IIS Manager, connect to the Application Request Routing server:

ARRConnect

When prompted, enter the Administrator username (in the format of ARRSERVERNAME\Username) and password for the Application Request Routing server.  Next, you'll be prompted to download and install the features to match the server:

![ARRIISFeatures](/assets/images/2013/05/ARRIISFeatures.png)

First thing is to edit the IIS Bindings of the Default Web Site, adding the SSL certificate that matches what is used on SharePoint.

![ARRIISBinding](/assets/images/2013/05/ARRIISBinding.png)

The next couple of settings will modify the DefaultAppPool to not timeout or recycle.  Using the IIS Manager, you can easily change these settings on the Advanced Settings and Recycling Conditions, respectively:

![AARIISAppPoolTimeout](/assets/images/2013/05/AARIISAppPoolTimeout.png)

![AARIISAppPoolRecycle](/assets/images/2013/05/AARIISAppPoolRecycle.png)

Optionally, this can be done from the Application Request Routing host via appcmd.exe:

```powershell
C:\Windows\System32\inetsrv\appcmd.exe set apppool "DefaultAppPool" /processModel.idleTimeout:00:00:00 /commit:apphost
C:\Windows\System32\inetsrv\appcmd.exe set apppool "DefaultAppPool" /recycling.periodicRestart.time:00:00:00 /commit:apphost
```

Finally, before we get to creating the Server Farm, add the OptionalWinHttpFlag via PoweShell on the Application Request Routing host:

```powershell
New-ItemProperty 'HKLM:\SOFTWARE\Microsoft\IIS Extensions\Application Request Routing' -Name OptionalWinHttpFlag -Value 72 -PropertyType Dword
```

Back in the IIS Manager, create a new Server farm named "SharePoint" (or anything you want it to be named).

![ARRIISCreateFarm](/assets/images/2013/05/ARRIISCreateFarm.png)

Add all SharePoint servers that respond directly to end user requests to the new farm.

![AARIISAddServer](/assets/images/2013/05/AARIISAddServer.png)

The Create Farm wizard will prompt if you want to create the appropriate URL rewrite rules.  Unless you have an advanced configuration, just say yes here.

Click on the farm name in the left hand tree.  Here you will see the options available to you to configure the farm.  One thing to immediately note is the Server Affinity feature.

![AARIISServerAff](/assets/images/2013/05/AARIISServerAff.png)

If you are using SharePoint 2010 or below, check Client affinity.  If you using SharePoint 2013, this is not required, but consider its use if not using SSL offloading as renegotiation of an SSL session is expensive.  Under the Routing Rules feature, disable SSL Offloading if you are not using it.

Implementation and testing of the ARR server is completely transparent to the user -- because you don't have to redirect user requests through the ARR prior to a production deployment in order to validate the configuration functions correctly.  Modify your client's hosts file (`C:\Windows\System32\drivers\etc\hosts`) with an entry similar to:

```text
# ARRServerIP SharePointHostname
192.168.1.10 sharepointwebapp
```

Next, from the client, navigate to the site.  If everything loads, great!  To validate that we are routing through our new reverse proxy, run Fiddler while browsing the site.

You'll see entries from both SharePoint and the IIS ARR module in the request and response headers, like this:ARRFiddler

Here we see the **X-Powered-By ARR/2.5** header as well as SharePoint's **MicrosoftSharePointTeamServices** and **X-SharePointHealthScore** header.  And no, this is not SharePoint 2010 or 2013 running on Windows Server 2012, the Server header comes from the ARR IIS8 server instead of the SharePoint server.

This should hopefully help you investigate alternative options from Microsoft for reverse proxy server.

Advanced installation options for the ARR include leveraging an IIS Shared Configuration which allows you to join multiple IIS ARR servers with identical configurations.  You must have a CIFS/SMB share available to store the configuration.  In addition, you can examine using Windows Network Load Balancing as a free option to balance requests between the IIS ARR servers (but it is highly recommended to investigate hardware load balancer alternatives).

The IIS ARR itself will do far more than I've outlined here.  For other features, take a look at the [IIS.Net Application Request Routing](http://www.iis.net/downloads/microsoft/application-request-routing) site!