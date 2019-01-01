---
layout: post
title: Integrating Microsoft SQL Reporting Services 2005 with SharePoint
tags: [sp2007, ssrs]
---

This guide assumes you already have a fully working farm, and a separate server where you will be installing SSRS (which already has SharePoint installed on it and is a member of the SharePoint farm).  It also assumes that you have a fully functional Server 2003 forest/domain and that all servers involved in the SharePoint farm are either using NetBIOS names for the URL or if using a dotted domain for the URL, are placed in the Intranet Zone in IE.  It also assumes that if you want to make SSRS available externally, the SSRS web site will also be available externally.

You will need [SetSPN](http://www.microsoft.com/downloads/details.aspx?displaylang=en&FamilyID=5fd831fd-ab77-46a3-9cfe-ff01d29e5c46) on a SharePoint Front End hosting the Central Administration and your to-be SSRS server.  You can place this utility in %WINDIR%\System32 for easy access.

First, you will need to use SetSPN to set up Kerberos authentication for your Central Administration site.  Your Central Administration and all SharePoint web sites should be using an Application Pool in IIS running as a Domain User account.  We will be using this account to enable Kerberos.

First, open a command prompt and type:

```text
setspn –A HTTP/centraladminsite DOMAIN\UserAccount
setspn –A HTTP/centraladminsite.domain.com DOMAIN\UserAccount
setspn –A HTTP/sharepointwebsite.domain.com DOMAIN\UserAccount
```

The output should show the registration of the SPN to the User account provided.

Open a Command Prompt and navigate to `%SYSTEMDRIVE%\Inetpub\AdminScripts`.  Type in:

```text
cscript //H:cscript
```

This will register `cscript` as your default scripting host.  Open the IIS Manager and locate the Web Site Identifier (unique for each web site).  In IIS6, you do this by expanding the server and clicking on Web Sites.  In the right hand pane, a list of Web Sites will appear with their identifier.

Back at the command prompt, for your Central Admin site, type in:

```text
adsutil.vbs set w3svc/WebSiteIdentifier/root/NTAuthenticationProviders “Negotiate,NTLM”
```

Note that “NTAuthenticationProviders” and “Negotiate,NTLM” is case-sensitive.  Repeat the above command for the SharePoint site that will host an SSRS report.

Back in the Command Prompt, navigate to `%programfiles%\Common Files\Microsoft Shared\web server extensions\12\BIN` and type in:

```text
stsadm –o setsharedwebserviceauthn –negotiate
```

This will set your Central Administration site to use Kerberos authentication.

Open Active Directory Users and Computers (ADUC), which is located under Administration Tools.  Find all SharePoint Servers involved in the farm (this includes your SQL Server, given your SQL Server does not have SharePoint installed on it).  For each server, open the Properties page and click on the Delegation tab.  Select the following option:

```text
Trust this computer for delegation to any service (Kerberos only)
```

Find the User Account running the Application Pool for Central Admin/SharePoint web sites.  Again go to Properties and find the Delegation tab.  Click on:

```text
Trust this user for delegation to any service (Kerberos only)
```

This change may take some time to replicate to all Domain Controllers.

In the mean time, download the [SharePoint Reporting Services add-on](http://www.microsoft.com/downloads/details.aspx?displaylang=en&FamilyID=1e53f882-0c16-4847-b331-132274ae8c84).  Install this add-on on each SharePoint Front End in the farm.

On the SSRS server, run the SQL Server 2005 installation.  For Components to Install, just select Reporting Services.  Install under the Default Instance name.  For the Service Account, use a Domain User Account.

It is highly recommended that you install [SQL Server 2005 Service Pack 3](http://www.microsoft.com/downloads/details.aspx?FamilyID=ae7387c3-348c-4faa-8ae5-949fdfbe59c4&displaylang=en)  (or later).  This Service Pack introduces [significant performance gains](http://support.microsoft.com/kb/955706) in SharePoint Integrated SQL Reporting Services.

Once completed, open the Reporting Services Configuration application.  For the Report Server Virtual Directory, use the default settings (the Default Web Site should be selected).  Repeat for the Report Manager Virtual Directory.

For the Web Service Identity, you should have a Domain User Account that will run SSRS (this should not be the same account as the SharePoint Application Pools use).  Create a new Application Pool and choose Windows Account.  Enter your Domain User Account.

Click on Database Setup and then enter the SQL Server that will host the ReportServer and ReportServerTemp databases.  Select New.  Check the box labeled “Create the report server database in SharePoint Integrated mode”.  After clicking OK, select “Windows Credentials” for Credentials Type.  Enter the same Domain User Account used in the Application Pool (e.g. DOMAIN\SSRSUser) and the user account’s password.  Continue with the remainder of the SSRS setup (Email Settings, Execution Account, etc.).

On the server hosting the Central Administration website, open up Computer Management.  Go to Local Users and Groups, then to Groups.  Open the WSS_WPG group and add the Domain User Account used in the SSRS setup.  Go back to the SSRS server and restart the SQL Reporting Server Reporting Services service.  In the Reporting Services Configuration Manager, click on Refresh.  A check box should appear next to the SharePoint Integration option on the left hand side.  Using SetSPN on the SSRS server, type in:

```text
setspn –A HTTP/ssrsservername.domain.com DOMAIN\SSRSUser
setspn –A HTTP/ssrsservername DOMAIN\SSRSUser
```

Open Active Directory Users and Computers and find the Domain User Account running SSRS.  Go to Properties and find the Delegation tab.  Click on:

Trust this user for delegation to any service (Kerberos only)

Go to the Central Administration website.  Go to the Application Management tab, there should be a new section named Reporting Services.  Click on “Manage integration settings”.  Enter the Reporting Server Web Service URL (e.g. http://ssrsservername.domain.com/reportserver).  Select Windows Authentication as the Authentication Mode.  Click OK.  Select “Grant database access”.  In the Server Name field, enter the SSRS server name (e.g. ssrsservername).  Select Default Instance (unless you specified the Instance Name during the SQL Setup for SSRS).  Click OK.  You will be prompted for Windows-based credentials.  This user account should have sa access on the SQL Server and local Administrator access on all servers in the SharePoint Farm.

Lastly, navigate to http://ssrsservername.domain.com/ReportServer.  If your User account and Computer are a member of the domain, you should not be prompted for credentials and you should see a Directory Listing.  Go back to the Application Management tab on the Central Administration site.  Under Reporting Services, click on “Reporting Services Server Defaults”.  You can adjust settings here if you wish, but you should not be prompted for credentials or receive any errors.  If this is the case, SSRS Integration works and Kerberos is fully functional with your Central Administration site.

To set up a SharePoint site with SSRS Integration, follow the SetSPN steps for each web site, getting each unique Web Site Identifier and running adsutil.vbs.  Finally, you will need to set the Authentication Mode for the SharePoint site to Kerberos.  To do this, go to the Application Management tab and under Application Security, select “Authentication providers”.  Select the Zone (usually “Default”).  Change your Web Application to the appropriate SharePoint site.  Under IIS Authentication Settings, choose “Negotiate (Kerberos)” and then click on Save.

Navigate to the SharePoint site (again, you should not be prompted for credentials).  Go to Site Actions and then Site Settings.  Under Site Collection Administration, click on “Site collection features”.  Click the Activate button next to “Report Server Integration Feature”.  Go back to the Site Settings and you will see a new section titled Reporting Services.  Click on “Manage Shared Schedules”.  You should be brought to a new page and no errors, nor prompts for credentials should occur.

That’s it!