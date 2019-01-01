---
layout: post
title: SharePoint 2010 and Active Directory Lightweight Directory Services – Better Together
tags: [ad-lds, sp2010]
---

Using [Active Directory Lightweight Directory Services](http://technet.microsoft.com/en-us/library/cc754361(WS.10).aspx) can have many advantages over using a SQL membership database, common to ASP.NET applications, including:

* Easy replication
* Active Directory Recycle Bin
* Easy application partitioning
* Inherits the password policy from the host server or if joined to a domain, Active Directory
* Wizard-based deployment
* Active Directory administrators can feel familiar with the tools and objects

This post assumes the following:

* SharePoint 2010 SP1 or higher is being used
* Dedicated server for AD LDS
* A trusted Certificate Authority is available
* Claims to Windows Token Service is functional on the SharePoint Server
* Accounts are set up per Todd Klindt’s Service Account Suggestions

AD LDS is deployed by installing the AD LDS Role on Server 2008 or Server 2008 R2. This “role” installation only installs the tools necessary to deploy an AD LDS instance, as well as the tools to manage the instance, ADSI Edit (`adsiedit.msc`) and ldp.exe.

To start, open Server Manager on the server that will be hosting the AD LDS instance(s). Add the Role, “Active Directory Lightweight Directory Services”, which if not already installed, will require the .NET Framework 3.5.1 Feature to be installed as well.

To install an instance of AD LDS, run the Active Directory Lightweight Directory Services Setup Wizard found in Administrative Tools. Install a unique instance:

![image_thumb222](/assets/images/2012/01/image_thumb222.png)

Name the instance and provide it a description. The name of the instance will be the display name of the Windows Service in services.msc, as well as the display name in Programs and Features.

![image_thumb242](/assets/images/2012/01/image_thumb242.png)

Provide a free LDAP and LDAPS (LDAP over SSL) port. These must be unused on the server by any other application (such as Active Directory, OpenLDAP, another AD LDS instance, etc).

![image_thumb252](/assets/images/2012/01/image_thumb252.png)

Next, decide whether or not to create an Application Partition. Application Partitions are where user, groups, and other objects are created. A single AD LDS instance can have many Application Partitions, or just one (or none, but that may not be of much use). If an Application Partition is not created now, one can be created later using ldp.exe.

The Partition name must be unique within the AD LDS instance. Also, the path cannot be a child of an existing path. For instance, if a Partition of “DC=fabrikam,DC=local” is created, another partition of “CN=MyApplication,DC=fabrikam,DC=local” cannot be created. However, two Partitions of “CN=MyApplication,DC=fabrikam,DC=local” and “CN="MyApplication2,DC=fabrikam,DC=local” can be created.

![image_thumb262](/assets/images/2012/01/image_thumb262.png)

The next steps involve setting the location of the AD LDS database and what account to use to perform AD LDS operations. This account is used to run the AD LDS instance. Add the logged in account, or any other account to the Administrative role of the AD LDS instance.

The last step is to decide which LDIF files to import. For SharePoint, and for AD LDS replication, import, at a minimum, the following LDIF files:

![image_thumb292](/assets/images/2012/01/image_thumb292.png)

Once the instance has been successfully created, connect to it with ADSI Edit, by running adsiedit.msc. Specify the Connection Point as the Partition name specified in the Instance creation wizard. Enter the AD LDS server and port number. By default, this is what will be in the Instance:

![image_thumb334](/assets/images/2012/01/image_thumb334.png)

Create a new Container object at the Partition root for User objects.

In CN=Roles, add the following Windows Accounts from Active Directory to the Readers group, referencing [Todd Klindt’s Service Account Suggestions](http://www.toddklindt.com/ServiceAccounts):

Farm Administrator: sp_farm

Application Pool Account that is running the Web Application leveraging AD LDS: sp_webapp

Search Crawl Account: sp_content

To do this, edit the Readers group and modify the “member” Attribute.

![image3](/assets/images/2012/01/image3.png)

![image8](/assets/images/2012/01/image8.png)

Click OK to add the above users to the member Attribute, and OK on the Readers role to finalize the change.

If the SharePoint application will be performing other administrative tasks on the Partition (e.g. allowing users to change passwords, delete accounts, or creating accounts), add the appropriate user to the Administrator group in the Partition.

The next step can be skipped, however, again if performing administrative tasks via SharePoint using the LDAP Membership Provider, and for general security purposes, leveraging SSL for the AD LDS instance is highly recommended.

Request a certificate with the Server Authentication purpose from either an internal Certificate Authority, or a Public Certificate Authority (the AD LDS and SharePoint Server(s) must trust the CA).  The Private Key must be exportable for this certificate.  An SSL-type certificate will work for this purpose.  The certificate should be stored in the Local Computer store.  Export the certificate as a PFX file.

Next, in the MMC, add the Certificates snap-in.  Select “Service account” for certificate management.  Find the service for the AD LDS instance (“Fabrikam”, in the above case).  Import the PFX into the Personal store.

Provide “Network Service” (or which ever account was specified to run the AD LDS instance, if it was not a Local Administrator) with Read & Execute NTFS permissions to:

`C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys\`

Note that the files beneath this folder do not inherit permission.  It may be necessary to grant Network Service (or the service account running the instance) with explicit access to the machine key for the requested certificate.

In `services.msc`, find the AD LDS instance service by name, and restart it.  This will allow the instance to bind to the appropriate certificate and provide LDAPS access.

Validate the LDAPS access via adsiedit.msc or ldp.exe.

![image14](/assets/images/2012/01/image14.png)

Enter the LDAPS port number and select “Use SSL-based Encryption”.

Note that if any changes are made to the AD LDS instance while ADSI Edit is running, such as setting up LDAPS, close ADSI Edit and reopen it before attempting to connect, otherwise an error may occur.

For ldp.exe, click on Connection –> Connect and enter:

![image17](/assets/images/2012/01/image17.png)

If no errors occur, then LDAPS was successfully set up.

Next, we move to attaching SharePoint to the AD LDS Partition.

Create a Claims-Based Web Application.  In the Claims Authentication Types, enable Forms Based Authentication and create new Membership and Role names.  These names will be used in web.configs.  If there are multiple Web Applications using one or more AD LDS Partitions or Instances, these names should be unique on a per-Web Application basis.

![image22](/assets/images/2012/01/image22.png)

Next, the following web.configs will need to be edited:

1. The Claims Based Web Application (on each WFE)
2. Central Administration
3. Security Token Service (on each SharePoint Server)

Because the below examples will not be done via the Object Model, the changes will not be saved to the SharePoint Configuration database.  This means the changes will not be replicated between WFEs, or if another WFE is added in the future, the change will not be present.  This can be resolved by creating a custom deployment via a SharePoint Solution, but that is outside of the scope of this article.

Also, because the below example is manually editing the web.configs, these web.configs must be manually backed up along side the standard SharePoint backup (<http://technet.microsoft.com/en-us/library/ee748650.aspx>).

First, edit the web.config of the Claims Based Web Application, making the following changes:

```xml
<PeoplePickerWildcards>
	<clear />
	<add key="AspNetSqlMembershipProvider" value="%" />
	<add key="FabrikamMember" value="*" />
</PeoplePickerWildcards>
```

Note the use of the asterisk, which is the wildcard character for LDAP searches, unlike the percent sign, which is used for T-SQL wildcard searches.

```xml
<membership defaultProvider="i">
	<providers>
		<add name="i" type="Microsoft.SharePoint.Administration.Claims.SPClaimsAuthMembershipProvider, Microsoft.SharePoint, Version=14.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c" />
		<add name="FabrikamMember" type="Microsoft.Office.Server.Security.LdapMembershipProvider, Microsoft.Office.Server, Version=14.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c" server="adlds01.nauplius.local" port="636" useSSL="true" userDNAttribute="distinguishedName" useDNAttribute="true" userNameAttribute="mail" userContainer="CN=SharePoint,DC=Fabrikam,DC=local" userObjectClass="user" userFilter="(ObjectClass=user)" scope="Subtree" otherRequiredUserAttributes="sn,givenname,cn" />
	</providers>
</membership>
<roleManager defaultProvider="c" enabled="true" cacheRolesInCookie="false">
	<providers>
		<add name="c" type="Microsoft.SharePoint.Administration.Claims.SPClaimsAuthRoleProvider, Microsoft.SharePoint, Version=14.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c" />
		<add name="FabrikamRole" type="Microsoft.Office.Server.Security.LdapRoleProvider, Microsoft.Office.Server, Version=14.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c" server="adlds01.nauplius.local" port="636" useSSL="true" groupContainer="CN=SharePoint,DC=fabrikam,DC=local" groupNameAttribute="cn" groupNameAlternateSearchAttribute="cn" groupMemberAttribute="member" userNameAttribute="mail" useUserDNAttribute="true" dnAttribute="distinguishedName" userFilter="&amp;(objectClass=user)(objectCategory=person)" groupFilter="&amp;(objectCategory=Group)(objectClass=group)" scope="Subtree" />
	</providers>
</roleManager>
```

Note the use of the Membership and Role provider names.  Also note the user/groupContainer Attributes, as well as the server, port, and useSSL attributes.  Lastly, note the use of the userNameAttribute specified as “mail”.  This means that all users created must have an Attribute of “mail” populated, and log into SharePoint with this Attribute.  Feel free to change this Attribute as required, e.g. to “cn”, or a custom attribute.

Next, change the Central Administration web.config, making the following changes:

```xml
<PeoplePickerWildcards>
	<clear />
	<add key="AspNetSqlMembershipProvider" value="%" />
	<add key="FabrikamMember" value="*" />
</PeoplePickerWildcards>

<roleManager enabled="true" defaultProvider="AspNetWindowsTokenRoleProvider">
	<providers>
		<add name="FabrikamRole" type="Microsoft.Office.Server.Security.LdapRoleProvider, Microsoft.Office.Server, Version=14.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c" server="adlds01.nauplius.local" port="636" useSSL="true" enableSearchMethods="true" groupContainer="CN=SharePoint,DC=Fabrikam,DC=local" groupNameAttribute="cn" groupNameAlternateSearchAttribute="cn" groupMemberAttribute="member" userNameAttribute="mail" dnAttribute="distinguishedName" useUserDNAttribute="true" scope="Subtree" userFilter="&amp;(objectClass=user)(objectCategory=person)" groupFilter="&amp;(objectCategory=Group)(objectClass=group)" />
	</providers>
</roleManager>
<membership>
	<providers>
		<add name="FabrikamMember" type="Microsoft.Office.Server.Security.LdapMembershipProvider, Microsoft.Office.Server, Version=14.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c" server="adlds01.nauplius.local" port="636" useSSL="true" enableSearchMethods="true" userDNAttribute="distinguishedName" userNameAttribute="mail" userContainer="CN=SharePoint,DC=Fabrikam,DC=local" userObjectClass="user" userFilter="(ObjectClass=*)" scope="Subtree" otherRequiredUserAttributes="sn,givenname,cn" />
	</providers>
</membership>
```

Lastly, edit the Security Token Service (STS) web.config.  This must be done on every server in the farm.  By default, the STS web.config is located at `C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\14\WebServices\SecurityToken\web.config`.

With the STS web.config, make this new section after the closing `</system.net>` tag, but before the closing `</configuration>` tag.

```xml
<system.web>
	<membership>
		<providers>
			<add name="FabrikamMember"
                  type="Microsoft.Office.Server.Security.LdapMembershipProvider, Microsoft.Office.Server, Version=14.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c"
                  server="adlds01.nauplius.local"
                  port="636"
                  useSSL="true"
                  enableSearchMethods="true" 
                  userDNAttribute="distinguishedName"
                  userNameAttribute="mail"
                  userContainer="CN=SharePoint,DC=Fabrikam,DC=local"
                  userObjectClass="user"
                  userFilter="(ObjectClass=*)"
                  scope="Subtree"
                  otherRequiredUserAttributes="sn,givenname,cn" />
		</providers>
	</membership>
	<roleManager enabled="true">
		<providers>
			<add name="FabrikamRole"
                  type="Microsoft.Office.Server.Security.LdapRoleProvider, Microsoft.Office.Server, Version=14.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c"
                  server="adlds01.nauplius.local"
                  port="636"
                  useSSL="true"
                  enableSearchMethods="true" 
                  groupContainer="CN=SharePoint,DC=Fabrikam,DC=local"
                  groupNameAttribute="cn"
                  groupNameAlternateSearchAttribute="cn"
                  groupMemberAttribute="member"
                  userNameAttribute="mail"
                  dnAttribute="distinguishedName"
                  useUserDNAttribute="true"
                  userFilter="&amp;(objectClass=user)(objectCategory=person)"
                  groupFilter="&amp;(objectCategory=Group)(objectClass=group)"
                  scope="Subtree" />
		</providers>
	</roleManager>
</system.web>
```

In these web.config changes for the Claims Based Web Application, Central Administration, and STS, I left out two properties that can be set for the Membership and Role provider.  The properties are “connectionUsername=” and “connectionPassword=”.  If these two options are not specified, SharePoint will use a Simple Bind to connect to AD LDS.  In order to allow a Simple Bind, additional steps must be taken within the AD LDS instance.

First, connect to AD LDS using ADSI Edit.  Connect to the Configuration Partition:

![image32](/assets/images/2012/01/image32.png)

Next, navigate to CN=Directory Services,CN=Windows NT,CN=Services,CN=Configuration and set the dsHeuristics value to 0000002001001 (<http://technet.microsoft.com/en-us/library/cc816788(WS.10).aspx>) on the Directory Services container.


Lastly, add “ANONYMOUS LOGON” to the Readers group in the Application Partition, just like what was done with the previous SharePoint accounts.

![image43](/assets/images/2012/01/image43.png)

Note that previous accounts added do not show their display name or container, only their Distinguished Name.  This is because those accounts are now Foreign Security Principals in the Application Partition and no longer have the necessary information to populate the display name.  This is normal.

To test the Simple Bind, use ldp.exe.  Connect to the AD LDS server with the correct port information, and use SSL, if configured.  Next, go to Connection –> Bind.  Select Simple Bind and leave the User and Password fields blank.  Go to View –> Tree and enter the Distinguished Name of the Application Partition (“CN=SharePoint,DC=Fabrikam,DC=local” in this case).  If the tree is viewable, SharePoint will be able to successfully read the Application Partition without having to specify a specific username and password for the connection.  Because this allows ANONYMOUS LOGON, it would be recommended to restrict connectivity to the AD LDS Instance from trusted hosts only.

If the above steps have been followed and the tree is not viewable, restart the AD LDS instance in services.msc.

Next, create a new User object with ADSI Edit.  Right click in then User container previously created, and specify New –> Object.  Select the User class.  Give it a Common Name.

![image25](/assets/images/2012/01/image25.png)

On the next screen, select More Attributes.  Specify any attribute as desired (e.g. givenName, sn), including mail, or the selected log on attribute.  After entering values for an Attribute, do not forget to hit Set.  When finished, the account will be created in a Disabled state.  The account must first have a password that complies with the Active Directory password policy (which AD LDS inherits by default, when the AD LDS server is joined to a domain), or local password policy (if the AD LDS server is not joined to a domain).  Right click the new account and select Set Password.

To enable the user, edit the new account.  Find the msDS-UserAccountDisabled Attribute and set it to False.  Save the changes to the account.

Next, after creating a new Site Collection for the Claims Based Web Application, log in with a Site Collection Administrator account.

Grant the AD LDS user access to the site.  Attempt to use a wildcard search (e.g. search for “john” and it should resolve):

![image47](/assets/images/2012/01/image47.png)

Attempt to sign in with the AD LDS account, using the mail attribute:

![image51](/assets/images/2012/01/image51.png)

If the AD LDS account is logged in, you have successfully set up SharePoint 2010 with Active Directory Lightweight Directory Services!

![image55](/assets/images/2012/01/image55.png)

If you see an error similar to the below in the Application Event Viewer, double-check each and every web.config for typos or mis-configurations.

```text
Log Name: Application
Source: Microsoft-SharePoint Products-SharePoint Foundation
Date: 1/2/2012 10:20:01 PM
Event ID: 6143
Task Category: General
Level: Critical
Keywords:
User: NAUPLIUS\s-sp2010farm
Computer: SHAREPOINT.nauplius.local
Description:
Cannot get Membership Provider with name FabrikamMember. The membership provider for  this process was not properly configured. You must configure the membership provider in the .config file for every SharePoint process.
```