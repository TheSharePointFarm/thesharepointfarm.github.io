---
layout: post
title: A Practical Guide to Implementing Incoming Email using the SharePoint Directory Management Service
tags: [sp2010,sp2013]
---

SharePoint 2010 and 2013 include functionality to implement Incoming Email into Libraries and enable SharePoint Groups to leverage an email address in Active Directory.  For Libraries, SharePoint allows users to submit items to the Library without having to visit SharePoint.  For SharePoint Groups, a Distribution Group is created with the SharePoint group's members.  This implementation guide covers a specific way of implementing SharePoint Incoming Email.  Various parts of this guide can be done differently, but this is my personal preferred way of implementing SharePoint Incoming Email.
Implementation

In order to leverage SharePoint incoming email and the Directory Management Service, at a minimum the Exchange Server 2003 schema extensions are required.  Note that an Exchange server is not required.  You can leverage a 3rd party Active Directory-integrated product as the mail server.

The first step to leveraging the DMS would be to extend the Active Directory schema.  These schema extensions are on the Exchange 2003, 2007, 2010, or 2013 media.  In general, to perform the extension, a user that is an Enterprise and Schema Administrator would run:

Exchange 2003:

```powershell
setup.com /forestprep
setup.com /domainprep
```

Exchange 2007 and higher:

```powershell
setup.exe /PrepareSchema
setup.exe /PrepareAD
setup.exe /PrepareDomain
```

There are various articles revolving other requirements surrounding each of the versions of Exchange's requirements for extending the schema, such as forest and domain level, etc.  For the purposes of this article, an Exchange 2010 SP2 server has been installed in the domain.

The next step is to identify one or more SharePoint Servers in the farm to handle Incoming Email as well as how to weight the SharePoint Servers within DNS using MX records.  If you want to use DNS to round-robin, each SharePoint Server will receive the same weight in the MX records.  If you want a primary and a fallback Incoming Mail setup, then weight one SharePoint Server with a lower value for MX (e.g. 10) and another one higher (e.g. 20).  The server with a lower value will always handle the Incoming Email unless it is unavailable.

First, create a new MX record in your domain.  For the purposes of this example, I used "sharepoint", so my FQDN would be "sharepoint.nauplius.local".  While there is only a single Incoming Email server in this example, the steps to set up each server are the same.  In this example, I'm routing "sharepoint.nauplius.local" (MX record) to "sharepoint.nauplius.local" (A record).  In a production environment, you might route "sharepoint.mycompany.com" (MX record) to "spserver01.mycompany.com" (A record).

![SharePointMXRecord](/assets/images/2013/02/SharePointMXRecord.png)

Once the MX record has been established, the next step is to install the IIS6 SMTP Service on your target SharePoint Server(s).  To do this, run the following PowerShell command or install the Simple Mail Transport Protocol Feature in Server Manager.

```powershell
Import-Module ServerManager
Add-WindowsFeature SMTP-Server
```

Next, configure the IIS6 SMTP service.  Under Administrative Tools, find Internet Information Services (IIS) 6.0 Manager.  Expand "[SMTP Virtual Server #1]" and "Domains".  Rename the default domain from `servername.mycompany.com` to match the MX record previously created.  In this example, `sharepoint.nauplius.local`.

![SMTPDomains](/assets/images/2013/02/SMTPDomains.png)

If you go to Properties on the domain, you can configure the Drop folder here if the default of `C:\inetpub\mailroot\drop` isn't a desirable location.  However, this Drop folder must be consistent on all SharePoint Servers.

Go to Properties on the SMTP Virtual Server #1.  Under Access -> Authentication, validate that Anonymous is checked (note that SharePoint handles authorization for incoming email, so Anonymous, for most installations, should be okay).

![SMTPAuthentication](/assets/images/2013/02/SMTPAuthentication.png)

Under Access -> Relay, place any Relay restrictions that are required, such as the Exchange Hub Transport server(s).

![SMTPRelay](/assets/images/2013/02/SMTPRelay.png)

Other settings you may want to consider are under the Messages tab.  While SharePoint will enforce the Web Application's maximum file size which has a default of 50MB, SMTP server's default mail size is significantly smaller at 2MB.  Consider changing the "Limit message size" and "Limit session size" values, or removing the restriction entirely.  The session size will be slightly larger than all messages in the session.

![SMTPMessages](/assets/images/2013/02/SMTPMessages.png)

For troubleshooting purposes and general logging, on the General tab click "Enable logging".  IIS SMTP logs are stored, by default, in `%windir%\system32\LogFiles\SMTPSVC1`.

The next step is to configure the Drop folder security.  Again, by default, this folder is located at `C:\inetpub\mailroot\drop`.  On the drop folder, give WSS_WPG NTFS Read & Execute rights.  Give `WSS_ADMIN_WPG` NTFS Full Control rights.  Do this for each SharePoint Server handling Incoming Email.

Now we're going to switch our attention to the Exchange Server to set up our Send Connector to cover the sharepoint.nauplius.local namespace.  Under Organization Configuration -> Hub Transport -> Send Connectors, create a new Send Connector.  Provide it a name, e.g. SharePoint. Leave the intended use as Custom.  In the Address Space screen, add an Address Space matching your SharePoint Incoming Email domain (in this example, sharepoint.nauplius.local).  All other settings can remain at their defaults.  For the Network Settings, we'll leverage MX to route email.  Add an applicable Source Server, or just use the default.  Here is an example of my setup using Exchange PowerShell:

```powershell
New-SendConnector -Name 'SharePoint' -Usage 'Custom' -AddressSpaces 'SMTP:sharepoint.nauplius.local;1' -IsScopedConnector $false -DNSRoutingEnabled $true -UseExternalDNSServersEnabled $false -SourceTransportServers 'EXCHANGE'
```

![SendConnector](/assets/images/2013/02/SendConnector.png)

Mail sent to sharepoint.nauplius.local will now route to the SharePoint Server.

The next step is to switch focus to Active Directory.  SharePoint will need an Organization Unit that it can create Contacts and Distribution Groups in.  This means we will need to delegate the SharePoint Timer Service (Database Access/Farm Admin) account rights on the selected Organization Unit in order to create, modify, and delete these objects.

First, create a new OU in an appropriate place in Active Directory.  In this example, I will be using nauplius.local/SharePoint/IncomingEmail (or the LDAP path of `CN=IncomingEmail,CN=SharePoint,DC=nauplius,DC=local`).

![OU](/assets/images/2013/02/OU.png)

Next, run the Delegate Control wizard on the new OU.  Delegate the access to the Timer Service account and create a custom task to delegate.  Delegate control of "Only the following objects in the folder".  Select "Contact objects" and "Group objects".  Also check the boxes for "Create selected objects in this folder" and "Delete selected objects in this folder".  Next, select Full Control.  This will give the Timer Service account Full Control over Contact and Group objects in this particular OU.  When looking at the Advanced Permissions on the OU after completing the delegation, it should look similar to:

![OU-AdvancedPermissions](/assets/images/2013/02/OU-AdvancedPermissions.png)

Back to Exchange, create a new Accepted Domain under Organization Configuration -> Hub Transport -> Accepted Domains.  Provide it with a name and the FQDN of the SharePoint domain (e.g. `sharepoint.nauplius.local`).  Set the type to Internal Relay as both Exchange and SharePoint will be handling email for this domain.

Create a new E-Mail Address Recipient Policy under Organization Configuration -> Hub Transport -> E-Mail Address Policies.  Provide the policy with a name that denotes that it is used with SharePoint.  Select the recipient container (Organization Unit) that contains the SharePoint-created Distribution Groups.  Include Mail-enabled groups only as the recipient type.  For the E-Mail Addresses, for the Local Part, use the alias, and for the domain, use the FQDN of the SharePoint domain (e.g. sharepoint.nauplius.local).  To create this in PowerShell, run:

```powershell
New-EmailAddressPolicy -Name 'SharePoint Policy' -RecipientContainer 'nauplius.local/SharePoint/IncomingEmail' -IncludedRecipients 'MailGroups' -Priority '1' -EnabledEmailAddressTemplates 'SMTP:%m@sharepoint.nauplius.local'
Update-EmailAddressPolicy -Identity 'SharePoint Policy'
```

Because SharePoint creates legacy Distribution Groups, create a PowerShell script on the Exchange server that runs the Update-EmailAddressPolicy command on a regular basis.  Distribution Groups that are not updated will not receive an email address and will be unable to be used by end users.

Let's shift focus to SharePoint.  Navigate to Central Administration.  Go into System Settings -> Configure incoming e-mail settings.

Under Enable sites on this server to receive incoming e-mail, select Yes.  Under Settings mode, select Advanced.  For "Use the SharePoint Directory Management Service to create distribution groups and contacts", select Yes.  Next, enter the LDAP path of the OU where the Timer Service has been delegated access to create Contact and Group objects.  Next, enter the SMTP namespace under "SMTP mail server for incoming mail".  In this example, that would be "sharepoint.nauplius.local".

If you only want users that are authenticated to be able to send email to SharePoint, select Yes on "Accept messages from authenticated users only".  Otherwise, select No.  This is also controlled at the Library level, although if you answer Yes here, the Library level will not be able to accept anonymous e-mail regardless of the Library setting.  Consider other 3rd party systems that may need to e-mail into SharePoint that are not tied into Active Directory when taking into account this particular setting.

If you want SharePoint to be able to create Distribution Groups, select Yes on "Allow creation of distribution groups from SharePoint sites".  If Yes is selected, the SharePoint Administrator can also require approval from a SharePoint Administrator prior to creating these Distribution Groups. Note that approving Distribution Groups must be done in a particular way.  Select any approvals that should be required in the following four checkboxes.

For the "Incoming E-Mail Server Display Address", enter the SMTP namespace that is used for SharePoint.  Again, in this example, it is "sharepoint.nauplius.local".

Finally, specify the E-Mail drop folder.  Again, by default it is "C:\inetpub\mailroot\drop".  This folder must be consistent on all SharePoint Servers handling Incoming Email.

![LibrarySettings](/assets/images/2013/02/LibrarySettings.png)

The last step to enabling Incoming Email is to start the Incoming Email service on all applicable SharePoint Servers in the farm.  Under Central Administration -> System Settings -> Manage services on server, start the "Microsoft SharePoint Foundation Incoming E-Mail" service.  This will kick off a timer job named "Microsoft SharePoint Foundation Incoming E-Mail" that runs once per minute that checks the Drop folder on the target server.  This timer job can either be monitored in the ULS logs, or in the Event Logs under Applications and Services Logs -> Microsoft -> SharePoint Products -> Shared -> Operational.  An Informational Event with an ID of 6871 with a Task Category of "E-Mail".  The event will log the number of messages processed, the time it took to process them, and the Message IDs.  It will also create an event if no messages are processed.

![EventLog](/assets/images/2013/02/EventLog.png)

Next, we move onto a SharePoint Site.  Again, each Library and SharePoint Group can have an associated email address.  For Libraries, go into the Settings for the Library and select "Incoming e-mail settings".  Here, you can enable Incoming Email into the Library. You can also configure the Local Part of the SMTP address, while Central Administration controls the Domain Part of the SMTP address.  If a Library attempts to leverage an already-in-use Local Part, it will indicate that there is a conflict and it that address cannot be used.  Other settings such as attachment handling, saving the original email, etc. can be configured here.  Lastly, you can configure whether or not incoming emails will be validated against the Library permissions (such as if a user emails a Library and does not have Contribute permission, their email will be rejected) or if it will accept email from any sender (this includes anonymous emails).

![LibrarySettings](/assets/images/2013/02/LibrarySettings.png)

Emails that are able to be authenticated (for example, a user using their Exchange account in the same Exchange Organization that SharePoint is a domain member of) will show the email having been created by that user.  Otherwise, the email will be created by the System Account.

Moving onto SharePoint Groups, go into the Site Settings -> People and Groups.  Under a Group that should be assigned an e-mail address, go to Settings -> Group Settings.  Under "Create an e-mail distribution group for this group", select Yes.  Provide a valid Local Part for the "Distribution list e-mail address".  If the SharePoint Administrator has required approval to create, modify, or delete a DL, then the SharePoint Group will sit in a Pending status until the SharePoint Administrator has approved the DL creation, modification, or deletion.  Otherwise, the action is processed immediately.

![GroupSettings](/assets/images/2013/02/GroupSettings.png)

If the SharePoint Administrator has required approval for creation, modification, or deletion of a Distribution Group for a SharePoint Group, then the Administrator will need to visit Central Administration -> Site Settings -> View All Site Content -> Distribution Groups.  Here they can view all pending requests and approve them.  However, please see Incoming Email Distribution Group Approval for the correct way to approve Groups, otherwise Groups will remain in a Pending status.
Troubleshooting

If Incoming Email is not working, there are a few things to validate.  First, validate that email can be sent to the server using the specified SMTP namespace by using telnet.  See KB153119 for details.  Next, check the SMTP service logs, by default located at %windir%\System32\LogFIles\SMTPSVC1.  RFC1893 defines the common SMTP status codes.  In general, 2xx is good, 4xx and 5xx are bad.  There is also the Event Log pointed out previously along with the ULS log.  The ULS logging for Incoming Email can be adjusted under Central Administration -> Monitoring -> Configure diagnostic logging -> SharePoint Foundation -> E-Mail.  Other things to check would be the Badmail and Queue directories in C:\inetpub\mailroot.  Also check the Exchange Queue Viewer for any errors when routing messages to your SharePoint SMTP namespace.

Note that any email rejected by the SharePoint timer job, for example due to lack of permissions on a Document Library or an invalid address, are immediately deleted.

**Fun Facts**

* The maximum alias length for a Distribution Group is 128 characters as is the maximum attachment file name length.
* If a Distribution Group is in the Pending state, Microsoft calls it a "Dirty DL".
* Contacts and Distribution Groups are created as Legacy when in Exchange 2007 or higher environments.  This is due to SharePoint not assigning an Exchange Version higher than "0".  See [SharePoint Creates Legacy Contacts in Active Directory]({% post_url 2012-04-10-sharepoint-creates-legacy-contacts-in-active-directory %}).


**Related Content**
[Setting Message Size Limits](http://technet.microsoft.com/en-us/library/cc785987(v=WS.10).aspx)

[Configure incoming e-mail (SharePoint Server 2010)](http://technet.microsoft.com/en-us/library/cc262947(v=office.14).aspx)

[Exchange Server 2010 Email Address Policies](http://exchangeserverpro.com/exchange-server-2010-email-address-policies)

[How to Create and Manage Exchange Server 2010 E-Mail Address Policies](http://answers.oreilly.com/topic/1240-how-to-create-and-manage-exchange-server-2010-e-mail-address-policies/)

[Incoming Email Distribution Group Approval]({% post_url 2012-11-13-incoming-email-distribution-group-approval %})

[SharePoint Creates Legacy Contacts in Active Directory]({% post_url 2012-04-10-sharepoint-creates-legacy-contacts-in-active-directory %})