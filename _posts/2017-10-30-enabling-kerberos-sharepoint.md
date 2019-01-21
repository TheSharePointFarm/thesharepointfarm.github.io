---
layout: post
title: Enabling Kerberos on SharePoint
tags: [sp2010,sp2013,sp2016]
---

I've seen a few guides on enabling Kerberos on SharePoint which are fairly poorly thought out, structured, or include unnecessary steps. Enabling your SharePoint Web Applications to use Kerberos is extremely simple and only requites two steps: Setting the SPN (Service Principal Name) on a Domain User account and enabling Kerberos on the Web Application.

First, identify the Domain User account used to drive the IIS Application Pool that is or will be assigned to your Web Application. In addition, you'll need the fully qualified domain name (you're not still using hostnames in 2017, right?) that your SharePoint Web Application will use.

Setting the SPN, by default, requires Domain Admin rights in Active Directory. This can be done from any computer and doesn't require you to be logged into a Domain Controller.

```
setspn -U -S HTTP/sharepoint.example.com CORP\webAppAccount
```

In this example, we're assigning an SPN to a user using the HTTP service (note we use 'HTTP' regardless if using http:// or https:// to access SharePoint). This service will be for the FQDN 'sharepoint.example.com' and our IIS Application Pool is driven by the webAppAccount user in the domain named CORP.

Once set (and perhaps waiting for Active Directory replication depending on the Active Directory environment), we need to then set the Web Application itself to use Kerberos. Note that this will cause a service interruption.

Navigate to Central Administration -> Manage Web Applications. Highlight the Web Application you wish to enable Kerberos, then click the Authentication button in the ribbon. Click on the zone (probably 'Default'). Scroll down to the Claims Authentication Types and select "Negotiate (Kerberos)". Click Save. Clicking save on this dialog will cause the Web Application to reprovision on all SharePoint servers where the Foundation Web service is started.

![KerberosAuthentication](/assets/images/2017/10/KerberosAuthentication.png)

Alternatively, this can be done via PowerShell to enable Kerberos.

```powershell
$wa = Get-SPWebApplication https://sharepoint.example.com
$wa.IisSettings[0].DisableKerberos = $false
$wa.Update()
$wa.ProvisionGlobally()
```

Once this process is complete, the Web Application will be set to use Kerberos.

A couple of notes on Kerberos: It won't be used in a scenario where the client cannot contact a Domain Controller; the client must be able to contact a DC in order to acquire a Kerberos ticket; for example, if the client is accessing SharePoint over the public Internet. You should never adjust Authentication methods or settings via IIS. SharePoint handles everything you need with regards to authentication on your Web Applications for you.

This guidance applies to SharePoint 2010 through 2016 in both Windows Classic and Windows Claims modes. The general guidance also applies to SharePoint 2007, but the method of setting Kerberos on the Web Application differs in Central Administration.