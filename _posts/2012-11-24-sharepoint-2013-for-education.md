---
layout: post
title: SharePoint 2013 for Education
tags: [sp2013]
---

Note: This was a feature in beta that was cut prior to RTM, however the bits remain in RTM.

This covers the installation process for SharePoint 2013 for Education using SharePoint 2013 Enterprise RTM.  This post does not cover any licensing/usability issues that may not yet be addressed by Microsoft.

This solution involves a Web Application named SharePoint Education (http://education).
In order to start this process, we need to use PowerShell. The following cmdlets are available:

```powershell
Install-SPEduSites
Add-SPEduUser
Add-SPEduClassMember
Add-SPEduClass
Remove-SPEduClassMember
Get-SPEduServiceSetting
Set-SPEduServiceSetting
```

While provisioning the SharePoint for Education services, the following prerequisites are required:

Web Application:

    A) A site collection at the root that the user running the cmdlet is a Site Collection Administrator of.

    B) Self-Service Site Creation is turned on for the Web Application.Service Applications:

Service Applications:

    A) Managed Metadata Service Application, including Full Control over the Service Application under Administrators.  The Managed Metadata Service Proxy must have all 4 check boxes checked in the Properties, including “This service application is the default storage location for column specific term sets” and “Consumes content types from the Content Type Gallery at ”.

    B) User Profile Service Application, including Full Control over the Service Application.  The User Profile Synchronization Service must also be in the Started state.  A MySite host must be specified that is part of the Education Web Application.

    C) A configured User Profile Synchronization connection to Active Directory.

    D) A Session State Service

    E) A Search Service

Site Collections:

    A) A Search Center site collection using the Basic or Enterprise Search template.

    B) A MySite host.

The Education Sites setup can either leverage the traditional or Host-Based Site Collection model, and it looks like the HBSC method is strongly preferred. For example, in the traditional model, Classes are created through Central Administration, which is potentially not desirable as this is commonly a task non-IT personnel would perform instead.  Also, specific User Profile properties for students are created under the UPA.  Lastly, a site at /sites/admin using the Tenant Administration site template is created even when not using the HBSC model!  Due to the setup time of Host-based Site Collections, this example leverages the classic Web Application model, but don’t hesitate to follow Spencer Harbar’s [SharePoint 2013 Multi-Tenancy guide](http://www.harbar.net/articles/sp2013mt.aspx) (which includes a link to the SharePoint 2010 guide, which is what will mostly need to be followed)

Run the following cmdlet to create the Education setup:

```powershell
Install-SPEduSites -WebApplication http://education -MySiteHost http://education/sites/mysites -SearchCenter http://education/sites/search
```

This process may take some time as it will be creating multiple sites, activating features, and so forth.

```powershell
Activating Feature: EduAdminPages to central Admin Site
Activating Feature: EduAdminLinks to central Admin Web
Creating: /sites/global Site Collection
Attempting to create site: http://education:80/sites/global
Activating Feature: EduShared to Institution Site
Activating Feature: EduInstitutionSiteCollection to Institution Site
Creating: /sites/admin Site Collection
Attempting to create site: http://education:80/sites/admin
Activating Feature: EduInstitutionAdmin to Admin Site
Creating: /sites/studygroup Site Collection
Attempting to create site: http://education:80/sites/studygroup
Activating Feature: Ratings to Study Group Site
Activating Feature: EduShared to Study Group Site
Activating Feature: EduCommunitySiteId to Study Group Site
Activating Feature: EduEduSearchDisplayTemplates to Study Group Site
Activating Feature: SearchWebPartFeature to Study Group Site
Activating Feature: SearchTemplatesandResources to Study Group Site
Finished Activating Fetures for Study Group Site
Creating: /sites/academiclibrary Site Collection
Attempting to create site: http://education:80/sites/academiclibrary
Activating Feature: DocMarketPlaceSampleData to Academic Library Site
Adding Academic Library Site to promoted sites view
Ensure current user has user profile
Successfully added Academic Library Site to promoted sites view
Added "All-Authenticated-Users" security group to "Site Visitors" group in Academic Library
Activating Feature: EduShared to My Site Host
Activating Feature: EduDashBoard to My Site Host
Activating Feature: EduMySiteHost to My Site Host
Activating Feature: EduSearchDisplayTemplates to My Site Host
Activating Feature: EduEduSearchDisplayTemplates to Search Center Site
```

I encountered a failure to provision the Academic Library (not shown above) due to a null value on the `taxonomySession.DefaultSiteCollectionTermStore`.

```powershell
Attempting to create site: http://education:80/sites/academiclibrary
Install-SPEduSites :
ProvisionTaxonomyColumn::taxonomySession.DefaultSiteCollectionTermStore is
null. Please make sure Managed Metadata service is turned on.
At line:1 char:1 + Install-SPEduSites -WebApplication http://education -MySiteHost
http://education ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
~~~
    + CategoryInfo : InvalidData: (Microsoft.Offic...ProvisionCmdlet:
   EduSiteProvisionCmdlet) [Install-SPEduSites], SPException
    + FullyQualifiedErrorId : Microsoft.Office.Education.Institution.PowerShell.EduSiteProvisionCmdlet
```

This was due to not having all four options on the Managed Metadata Service Proxy checked in the Properties:

![image3](/assets/images/2012/11/image3.png)

This completes the installation process for Education sites.

Within General Settings, new options are added to Central Administration:

![image9](/assets/images/2012/11/image9.png)

A few new User Profile properties are created and listed under the Custom section:

```text
Role [“EduUserRole”] (integer): None (0), Educator (1), Student (2), Parent (3), TeacherAssistant (4), Administrator (5)
PersonalSiteState [“EduPersonalSiteState”] (integer): Unknown (0), Queued (1), Provisioned (2)
OAuthTokenProviders [“EduOAuthTokenProviders”] (string, single value, 2500 characters maximum)
SIS UserId [“SISUserId”] (string, single value, 400 characters maximum)
EduExternalSyncState [“EduExternalSyncState”]: None (0), ExchangeOauthCached (1), Unknown (2)
```

The Exchange features of Education requires Exchange 2013, unfortunately.  There looks to be integration with SiteMailboxes as well as Exchange calendaring functionality.  Education also has Lync integration, but I was unable to find a minimum version requirement.

From the Academic Library, an author can publish documents relevant to the scope of the Host-Based Site Collection or Web Application.  Documents are uploaded, tagged, etc:

![image14](/assets/images/2012/11/image14.png)

After a document is published, it allows users to rate the document.

From the Tenant Administration/Central Administration, Study Groups and Classes can be created.  For example, to create a Class, click Education in the ribbon and select the appropriate option.  Here is an example of Class creation:

![image18](/assets/images/2012/11/image18.png)

Study Groups are created in the same way.  Study Groups look to be very similar to the old Meeting workspaces in function: there is a Document repository, a Group discussion, Group notes (a pre-created OneNote document resides here), and a Group Calendar.

Classes, Memberships, and Users appear to be created via the one Timer Job installed by the Education functionality, Education Bulk Operation.  By default, this runs every 5 minutes.

Microsoft included a Data Import and Export functionality from the Administration site.  From here, you can update user, class, and membership information.  The type of operation (add, update, delete) as well as type of entity (user, class, membership) appears to be validated based on the format of the header row of the CSV.  For example, the following CSV will create two new classes:

[SharePointCourses](/assets/other/2012/11/SharePointCourses.csv)

On the Results and Reports page, we can see I got the last one right!  You’ll also note that you can upload the same file name and SharePoint will automatically generate a new name for it if an existing file is already present.

![image31](/assets/images/2012/11/image31.png)

Again, this is driven by the Education Bulk Operation timer job.

![image27](/assets/images/2012/11/image27.png)

One thing to note is that imports will fail unless you have added the UserId as an Education user.  For example, I would run the following command to set myself as an Educator:

```powershell
Add-SPEduUser -UserAlias "nauplius\trevor" -Site http://education -Role "Educator"
Add-SPEduUser will only accept Educator and Student, but none of the other roles as defined by the new Role User Profile property.
```

There appears to be a lot of functionality in the Education section of SharePoint.  I’ve only touched on the basic Administrator-level functionality.  This appears to be a great replacement for those still using Microsoft Class Server 4.0 or the SharePoint Learning Kit.