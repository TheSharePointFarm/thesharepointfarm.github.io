---
layout: post
title: Using an External Identity Manager for SharePoint User Profile Synchronization
tags: [sp2010,sp2013]
---

Forefront Identity Manager 2010 R2 SP1 and SharePoint Server 2013 has introduced the ability to leverage FIM for User Profile Synchronization with Active Directory, versus the built-in version of FIM included with SharePoint Server.

The SharePoint Connector officially supports SharePoint Server 2013, but will unofficially work with SharePoint Server 2010.

You will need a few Domain accounts.  An account to run the FIM Service (s-fim), an account to run the FIM Management Agents (s-fimma), the SharePoint farm administrator account (s-sp2013farm), and finally a synchronization account for Active Directory (s-sp2013sync).  For the last account, this guide will be using the same account as the one used for the UPA connection.  Configure the permissions appropriately for s-sp2013sync.

Provision the UPA and UPSS per the standard instructions.  Once both services have been configured, stop the FIM services on the SharePoint server and set them to Disabled.  In the UPA under Configure Synchronization Settings, you have Enable External Identity Manager selected.

![UPA1](/assets/images/2013/05/UPA1.png)

First, we’ll start out with a SQL Server running SQL Server 2012 SP1 with the Database Engine, Integration Services, and Management Studio.  All other settings are at their defaults.  If you are using a SQL Server that is not running on the same server as the FIM services, make sure to install the SQL Server Native Client on the server running the FIM services.

The FIM server will run SharePoint Foundation 2013, the FIM Synchronization Service as well as FIM Service and Portal, along with the SharePoint User Profile Connector.

Install SharePoint Foundation 2013 and create a Classic Web Application for the FIM Portal.  The FIM Portal does not currently work with Claims-based Authentication.  Next, install the FIM Synchronization Service.  During the installation, specify the FIM Synchronization Service account.

![FIMSync1](/assets/images/2013/05/FIMSync1.png)

Next, install the FIM Service and Portal.  The Portal will leverage our SharePoint Foundation installation and [Classic Web Application](http://technet.microsoft.com/en-us/library/gg276326.aspx).  The Classic Web Application has been configured with an Alternate Access Mapping of “FIM02” in this example.

![FIMService1](/assets/images/2013/05/FIMService1.png)

Enter the SharePoint site collection URL.

![FIMService8](/assets/images/2013/05/FIMService8.png)

Enter the hostname of the FIM Service server.  We’re installing the Portal and Service on the same server, so again we’ll use “FIM02” here.

![FIMService7](/assets/images/2013/05/FIMService7.png)

Enter the hostname of the Synchronization Service, along with the Management Agent account.

![FIMService6](/assets/images/2013/05/FIMService6.png)

Again, enter the FIM Service service account information.

![FIMService5](/assets/images/2013/05/FIMService5.png)

You can either let FIM generate a self-signed certificate, or use a certificate signed by a Certificate Authority.  For purposes of synchronization, a self-signed certificate will work.

![FIMService4](/assets/images/2013/05/FIMService4.png)

Enter the mail server information.  Since we’re just after synchronization, the remaining options are unchecked (leaving the polling option checked, if not configured properly, will generate Event Log warnings).

![FIMService3](/assets/images/2013/05/FIMService3.png)

Enter the database server name and database name.

![FIMService2](/assets/images/2013/05/FIMService2.png)

Finish the installation of the FIM Service and Portal.  Next, install the [KB2832389](http://support.microsoft.com/kb/2832389) update for the FIM Synchronization Service and FIM Portal and Service.  This update is required prior to installing the SharePoint User Profile Connector.  Download the [Connector for SharePoint User Profile Store](http://www.microsoft.com/en-us/download/details.aspx?id=41164).  Install the SharePoint User Profile Connector on the FIM Synchronization Service server.

The next step is to use the FIM client and FIM Portal to set up our Management Agents, Synchronization Rules, Workflows, and Management Policy Rules.  This will cover the basics required, but you will want to adjust the attributes used and users targeted based on business requirements.  Lastly, this will only cover User objects, but Contact and Group objects are also available for synchronization.

First, let’s add a new attributes that we’ll use.  Using the Synchronization Service client, under the Metaverse Designer, select the person object type.  Create one attribute:

Attribute name: sAMAccountName

Attribute type: String (non-indexable)

Next, create the Management Agents.  Create a new Active Directory Domain Services MA.  Go through the Management Agent, enter the appropriate information.  For the username to connect to AD DS, specify the same account used for the User Profile Application connection (e.g. s-sp2013sync).  Select the Directory Partition as well as specify any Containers (or all Containers) you want to synchronize objects from.  Under object types, make sure at least User objects are selected.  Under Attributes, select:

```text
displayName, givenName, mail, objectSid, sAMAccountName, sn, telephoneNumber
```

Click Next until you complete the Management Agent.

Create the FIM Service Management Agent.  For this agent, under Connect to database, specify the values used to connect to the FIM Service.  In this example, the values are:

Server: localhost

Database: FIMService

FIM Service base address: http://localhost:5725

Using Windows Authentication, specify the FIM Service Management Agent account (not the FIM Service account):

User name: s-fimma

Password: <password>

Domain: nauplius

Under Object Types, make sure the Person, and optionally Group, object type is selected.  All Attributes should be selected.  Configure the Person Object Type Mapping to map from “Person” to “person”.  This is the only Management Agent where we will configure the Attribute Flow.  In this example, the flow is configured with these values:

![FIMMA1](/assets/images/2013/05/FIMMA1.png)

Click Next until you complete the Management Agent.

The last Management Agent we will create is the SharePoint Profile Store Management Agent.  Under Connectivity, specify the hostname and port number of the server running Central Administration.  Enter the domain credentials of the SharePoint farm administrator account.  For the picture flow directly, we are going to select “Export only (NEVER from SharePoint)”.  This will flow pictures from Active Directory to SharePoint.  Select all 3 Object Types.  This Management Agent will throw errors when attempting to synchronize with SharePoint if any of the object types are left deselected.  On the Attributes, select at least the following:

```text
AccountName, Anchor, domain, FirstName, LastName, Picture, PreferredName, ProfileIdentifier, SID, UserName
```

You may also add other attributes, such as WorkEmail, WorkPhone, and so forth.  This example will use some of these other attributes later in the Synchronization Rules.  Complete the SharePoint Management Agent.

If you export pictures from Active Directory to SharePoint, make sure you run the following on the SharePoint server:

```powershell
Update-SPProfilePhotoStore -MySiteHostLocation http://mysitehost -CreateThumbnailsForImportedPhotos $true
```

Configure Run Profiles for each Management Agent.  The Active Directory Management Agent requires Full Import, Full Synchronization, Delta Import, and Delta Synchronization.  The FIM Management Agent requires Full Import, Full Synchronization, Export, Delta Import, Delta Synchronization.  The SharePoint Management Agent requires Full Synchronization, Export, Delta Synchronization.

In the Synchronization Service Manager, go to Tools -> Options and select Enable Synchronization Rule Provisioning.

This completes the Management Agent setup.

Prior to creating the Synchronization Rules, you will want to run a PowerShell script to determine the Custom Expression for the domain attribute.  You can get the PowerShell script from [Using PowerShell To Generate The Custom Expression For The Domain Attribute Flow](http://social.technet.microsoft.com/Forums/en-US/ilm2/thread/50088024-d86a-49dc-bb03-3243ebd677eb).  Edit the variables at the beginning of the script to match your domain and forest.  Next, save the output value.

The next step will be to leverage the FIM Portal to configure the Synchronization Rules.  In our example, navigate to http://fim02/IdentityManagement.  In the left hand bar, click Administration, then Synchronization Rules.  Create a new rule.  This rule will be for Active Directory Import.  Provide it with an appropriate Display Name.  The Data Flow Direction is Import.  Set the Scope options for the Metaverse Resource Type to “person”, using the Active Directory Domain Services Management Agent, and the External System Resource Type of “user”.  Create a Relationship of accountName equal to sAMAccountName.  Select the option to Create resource in FIM.  Create the following Inbound Flows:

```text
displayName => displayName; givenName => firstName; objectSid => objectSid => sAMAccountName => accountName; sn => lastName; mail => email; telephoneNumber => officePhone
```

![FIMSyncRule1](/assets/images/2013/05/FIMSyncRule1.png)

Complete the creation of this particular Synchronization Rule.  Next, create an outbound Synchronization Rule for SharePoint.  Provide it with a Display Name, set the Data Flow Direction to Outbound, and Apply Rule to “To specific metaverse resources…”.  Under Scope, select “person”, the SharePoint Management Agent, and “user”.  Here we’ll set the Relationship to sAMAccountName equal to UserName.  Select the options to Create resource in external system and Disconnect FIM resource.  For the Outbound Attribute Flow, configure the flows similar to this:

```text
accountName => UserName; IIF(Eq(Left(ConvertSidToString(objectSid),40),”S-1-5-21-X-X-X”),”NAUPLIUS”,”Unknown”) => domain; IIF(Eq(Left(ConvertSidToString(objectSid),40),”S-1-5-21-X-X-X”),”NAUPLIUS”,”Unknown”)+”\”+accountName => AccountName; firstName => FirstName*; lastName => LastName*; IIF(Eq(Left(ConvertSidToString(objectSid),40),”S-1-5-21-X-X-X”),”NAUPLIUS”,”Unknown”)+”\”+accountName => ProfileIdentifier; firstName+” “+lastName => PreferredName*; email => WorkEmail*; officePhone => OfficePhone*; ConvertSidToString(objectSid)** => dn; ConvertSidToString(objectSid)** => Anchor.
```

Values marked with an * may have the option “Allow null attributes” checked and values marked with ** are set to Initial Flow Only.

![FIMSyncRule2](/assets/images/2013/06/FIMSyncRule2.png)

Note: If you set the Synchronization Rule to Apply Rule “to all metaverse resources of this type…”, you can skip creating a Workflow, MPR, and Set.  However, you do lose further flexibility by doing so.  For standard import purposes, this flexibility is likely not required.

Next, create a Workflow.  On the left, under Management Policy Rules, select Workflows.  This workflow will be used to export objects into SharePoint.  Name the workflow appropriately.  Leave Run on Policy Update unchecked.  For the Activity, add a new Synchronization Rule Activity type, then select the SharePoint Management Agent, and select Add as the Action Selection.  Save and exit the workflow.

![FIMWorkflow1](/assets/images/2013/05/FIMWorkflow1.png)

Create the Management Policy Rule.  Name the rule appropriately.  Select Transition Set, then select Transition In with an appropriate Transition Set. I will be using “All People” in this example.  Under the Policy Workflows, select the workflow you just created.

![FIMMPR1](/assets/images/2013/06/FIMMPR1.png)

The last step of the process is to run the Management Agents.  Run them in the following order:

**AD DS MA:**

Full Import

Full Synchronization

**FIM MA:**

Full Import

Full Synchronization

Export

Delta Import

**SharePoint MA:**

Full Synchronization

Export

Check the User Profile Service Application.  There should be a number of User Profiles.

On subsequent runs, use the following order:

**AD DS MA:**

Delta Import

Delta Synchronization

**SharePoint MA:**

Delta Synchronization

Export

If there is a change in the Management Agent rules, re-run the initial synchronization.  On the Configure Run Profiles window for each Management Agent, you can Script each run profile to VBScript.  This will allow you to create a scheduled task to execute the run profile automatically.

That is all there is to it.  Note that when upgrading SharePoint (Cumulative Update or Service Pack), you must re-provision the User Profile Synchronization Service, then stop it again.  This is required in order to update the User Profile databases.

Leveraging FIM as your Identity Manager will allow you to create more complex business rules, as well as import from a variety of sources not supported directly via SharePoint, or write your own custom code for import purposes.

Here are some helpful articles on FIM synchronization:

* [How Do I Synchronize Users from Active Directory Domain Services to FIM](http://social.technet.microsoft.com/wiki/contents/articles/how-do-i-synchronize-users-from-active-directory-domain-services-to-fim.aspx) (TechNet Wiki)
* [How do I Provision Users to Active Directory Domain Services](http://social.technet.microsoft.com/wiki/contents/articles/how-do-i-provision-users-to-active-directory-domain-services.aspx) (TechNet Wiki)
* [Custom expressions in advanced attribute flows](http://blogs.technet.com/b/doittoit/archive/2008/08/07/if.aspx)
* [Introduction to User and Group Management](http://technet.microsoft.com/en-us/library/ee534902(v=ws.10).aspx) (TechNet)