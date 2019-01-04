---
layout: post
title: December 2012 patch for SharePoint 2013 breaks MySite Site Creation
tags: [sp2013]
---

Todd Klindt brought this one to my attention.  The December 2012 patch (KB2752058) breaks the SharePoint 2013 MySite Self-Service Site creation, with an error similar to the below in the Event Log:

```text
Log Name:      Application
Source:        Microsoft-SharePoint Products-SharePoint Portal Server
Date:          1/3/2013 11:04:29 AM
Event ID:      5187
Task Category: Administration
Level:         Error
Keywords:      
User:          NAUPLIUS\s-sp2013farm
Computer:      SHAREPOINT2013.nauplius.local
Description:
My Site creation failure for user 'NAUPLIUS\trevor' for site url 'http://mysites2013/personal/trevor'. The exception was: System.MissingMethodException: Method not found: 'Microsoft.SharePoint.SPSite Microsoft.SharePoint.SPSite.SelfServiceCreateSite(System.String, System.String, System.String, UInt32, Int32, Int32, System.String, System.String, System.String, System.String, System.String, System.String, System.String, System.String, Microsoft.SharePoint.SPSiteSubscription)'.
   at Microsoft.Office.Server.UserProfiles.UserProfile.<>c__DisplayClass2.<CreateSite>b__0()
   at Microsoft.SharePoint.SPSecurity.<>c__DisplayClass5.<RunWithElevatedPrivileges>b__3()
   at Microsoft.SharePoint.Utilities.SecurityContext.RunAsProcess(CodeToRunElevated secureCode)
   at Microsoft.SharePoint.SPSecurity.RunWithElevatedPrivileges(WaitCallback secureCode, Object param)
   at Microsoft.SharePoint.SPSecurity.RunWithElevatedPrivileges(CodeToRunElevated secureCode)
   at Microsoft.Office.Server.UserProfiles.UserProfile.CreateSite(String strRequestUrl, Boolean bCollision, Int32 overrideCompatLevel, Int32 lcid).
```text

Additionally, this information is outputted in the ULS log:

```text
01/03/2013 11:04:29.00 	OWSTIMER.EXE (0x0780)                   	0x1844	SharePoint Portal Server      	User Profiles                 	aj1lz	Medium  	UserProfile.UpdatePersonalSiteCapabilities Updating value for user: NAUPLIUS\trevor value: None	983ff19b-0475-c0fa-03a6-fff304c21dba
01/03/2013 11:04:29.10 	OWSTIMER.EXE (0x0780)                   	0x1844	SharePoint Portal Server      	Personal Site Instantiation   	ogso	Medium  	[Processing] CreatePersonalSite: Testing for personal site: (http://mysites2013/personal/trevor) for user: (NAUPLIUS\trevor).	983ff19b-0475-c0fa-03a6-fff304c21dba
01/03/2013 11:04:29.10 	OWSTIMER.EXE (0x0780)                   	0x1844	SharePoint Portal Server      	Personal Site Instantiation   	ogup	Medium  	[Processing] Attempting to create personal site at (http://mysites2013/personal/trevor) for user: (NAUPLIUS\trevor).	983ff19b-0475-c0fa-03a6-fff304c21dba
01/03/2013 11:04:29.10 	OWSTIMER.EXE (0x0780)                   	0x1844	SharePoint Portal Server      	Administration                	5187	Critical	My Site creation failure for user 'NAUPLIUS\trevor' for site url 'http://mysites2013/personal/trevor'. The exception was: System.MissingMethodException: Method not found: 'Microsoft.SharePoint.SPSite Microsoft.SharePoint.SPSite.SelfServiceCreateSite(System.String, System.String, System.String, UInt32, Int32, Int32, System.String, System.String, System.String, System.String, System.String, System.String, System.String, System.String, Microsoft.SharePoint.SPSiteSubscription)'.     at Microsoft.Office.Server.UserProfiles.UserProfile.<>c__DisplayClass2.<CreateSite>b__0()     at Microsoft.SharePoint.SPSecurity.<>c__DisplayClass5.<RunWithElevatedPrivileges>b__3()     at Microsoft.SharePoint.Utilities.SecurityContext.RunAsProcess(CodeToRunElevated secureCode)     at Microsoft.SharePoint.SPSecurit...	983ff19b-0475-c0fa-03a6-fff304c21dba
01/03/2013 11:04:29.10*	OWSTIMER.EXE (0x0780)                   	0x1844	SharePoint Portal Server      	Administration                	5187	Critical	...y.RunWithElevatedPrivileges(WaitCallback secureCode, Object param)     at Microsoft.SharePoint.SPSecurity.RunWithElevatedPrivileges(CodeToRunElevated secureCode)     at Microsoft.Office.Server.UserProfiles.UserProfile.CreateSite(String strRequestUrl, Boolean bCollision, Int32 overrideCompatLevel, Int32 lcid).	983ff19b-0475-c0fa-03a6-fff304c21dba
01/03/2013 11:04:29.10 	OWSTIMER.EXE (0x0780)                   	0x1844	SharePoint Portal Server      	Personal Site Instantiation   	ogsj	High    	Exception while creating personal site: () for (NAUPLIUS\trevor): (System.MissingMethodException: Method not found: 'Microsoft.SharePoint.SPSite Microsoft.SharePoint.SPSite.SelfServiceCreateSite(System.String, System.String, System.String, UInt32, Int32, Int32, System.String, System.String, System.String, System.String, System.String, System.String, System.String, System.String, Microsoft.SharePoint.SPSiteSubscription)'.     at Microsoft.Office.Server.UserProfiles.UserProfile.<>c__DisplayClass2.<CreateSite>b__0()     at Microsoft.SharePoint.SPSecurity.<>c__DisplayClass5.<RunWithElevatedPrivileges>b__3()     at Microsoft.SharePoint.Utilities.SecurityContext.RunAsProcess(CodeToRunElevated secureCode)     at Microsoft.SharePoint.SPSecurity.RunWithElevatedPrivileges(WaitCallback secureCode, Ob...	983ff19b-0475-c0fa-03a6-fff304c21dba
01/03/2013 11:04:29.10*	OWSTIMER.EXE (0x0780)                   	0x1844	SharePoint Portal Server      	Personal Site Instantiation   	ogsj	High    	...ject param)     at Microsoft.SharePoint.SPSecurity.RunWithElevatedPrivileges(CodeToRunElevated secureCode)     at Microsoft.Office.Server.UserProfiles.UserProfile.CreateSite(String strRequestUrl, Boolean bCollision, Int32 overrideCompatLevel, Int32 lcid)     at Microsoft.Office.Server.UserProfiles.UserProfile.CreatePersonalSiteWithSQM(String mySitePortalUrl, Int32 overrideCompatLevel, Int32 lcid)).	983ff19b-0475-c0fa-03a6-fff304c21dba
01/03/2013 11:04:29.10 	OWSTIMER.EXE (0x0780)                   	0x1844	SharePoint Portal Server      	Personal Site Instantiation   	ajjul	Exception	Exception during creation of personal site from MySiteInstantiationWorkItemJobDefinition::ProcessWorkItemCore() for NAUPLIUS\trevor System.MissingMethodException: Method not found: 'Microsoft.SharePoint.SPSite Microsoft.SharePoint.SPSite.SelfServiceCreateSite(System.String, System.String, System.String, UInt32, Int32, Int32, System.String, System.String, System.String, System.String, System.String, System.String, System.String, System.String, Microsoft.SharePoint.SPSiteSubscription)'.     at Microsoft.Office.Server.UserProfiles.UserProfile.<>c__DisplayClass2.<CreateSite>b__0()     at Microsoft.SharePoint.SPSecurity.<>c__DisplayClass5.<RunWithElevatedPrivileges>b__3()     at Microsoft.SharePoint.Utilities.SecurityContext.RunAsProcess(CodeToRunElevated secureCode)     at Microsoft.SharePoint...	983ff19b-0475-c0fa-03a6-fff304c21dba
01/03/2013 11:04:29.10*	OWSTIMER.EXE (0x0780)                   	0x1844	SharePoint Portal Server      	Personal Site Instantiation   	ajjul	Exception	....SPSecurity.RunWithElevatedPrivileges(WaitCallback secureCode, Object param)     at Microsoft.SharePoint.SPSecurity.RunWithElevatedPrivileges(CodeToRunElevated secureCode)     at Microsoft.Office.Server.UserProfiles.UserProfile.CreateSite(String strRequestUrl, Boolean bCollision, Int32 overrideCompatLevel, Int32 lcid)     at Microsoft.Office.Server.UserProfiles.UserProfile.CreatePersonalSiteWithSQM(String mySitePortalUrl, Int32 overrideCompatLevel, Int32 lcid)     at Microsoft.Office.Server.UserProfiles.UserProfile.CreatePersonalSite(Int32 lcid)     at Microsoft.Office.Server.UserProfiles.MySiteInstantiationWorkItemJobDefinition.ProcessWorkItemCore(SPWorkItemCollection workItems, SPWorkItem workItem, MySiteInstantiationWorkItemJobState timerJobState) StackTrace:  at Microsoft.Office.Server...	983ff19b-0475-c0fa-03a6-fff304c21dba
01/03/2013 11:04:29.10*	OWSTIMER.EXE (0x0780)                   	0x1844	SharePoint Portal Server      	Personal Site Instantiation   	ajjul	Exception	....Native.dll: (sig=1f86b0bf-2440-4b16-9099-860a571153c2|2|microsoft.office.server.native.pdb, offset=131CE) at Microsoft.Office.Server.Native.dll: (offset=21C85)	983ff19b-0475-c0fa-03a6-fff304c21dba
01/03/2013 11:04:29.10 	OWSTIMER.EXE (0x0780)                   	0x1844	SharePoint Server             	Unified Logging Service       	c91s	Monitorable	Watson bucket parameters: SharePoint Server 2013, ULSException14, 06d8f9f3 "sharepoint portal server", 0f001144 "15.0.4420.0", 294f4357 "kernelbase.dll", 060223f0 "6.2.9200.0", 5010ab2d "wed jul 25 19:27:57 2012", MISSING, 000189cc "000189cc", e0434352 "e0434352", 0024950b "ajjul"	983ff19b-0475-c0fa-03a6-fff304c21dba
01/03/2013 11:04:29.10 	OWSTIMER.EXE (0x0780)                   	0x1844	SharePoint Portal Server      	Personal Site Instantiation   	ajjun	Unexpected	MySiteInstantiationJob:ProcessWorkItem: ProcessWorkItemCore for My Site Instantiation for: 'NAUPLIUS\trevor' failed. Error Message: Method not found: 'Microsoft.SharePoint.SPSite Microsoft.SharePoint.SPSite.SelfServiceCreateSite(System.String, System.String, System.String, UInt32, Int32, Int32, System.String, System.String, System.String, System.String, System.String, System.String, System.String, System.String, Microsoft.SharePoint.SPSiteSubscription)'.	983ff19b-0475-c0fa-03a6-fff304c21dba
01/03/2013 11:04:29.10 	OWSTIMER.EXE (0x0780)                   	0x1844	SharePoint Server             	General                       	aeyvf	Unexpected	My Site Instantiation Interactive Request Queue - Failed to process a work item. This work item will not be retried. Error: System.MissingMethodException: Method not found: 'Microsoft.SharePoint.SPSite Microsoft.SharePoint.SPSite.SelfServiceCreateSite(System.String, System.String, System.String, UInt32, Int32, Int32, System.String, System.String, System.String, System.String, System.String, System.String, System.String, System.String, Microsoft.SharePoint.SPSiteSubscription)'.     at Microsoft.Office.Server.UserProfiles.UserProfile.<>c__DisplayClass2.<CreateSite>b__0()     at Microsoft.SharePoint.SPSecurity.<>c__DisplayClass5.<RunWithElevatedPrivileges>b__3()     at Microsoft.SharePoint.Utilities.SecurityContext.RunAsProcess(CodeToRunElevated secureCode)     at Microsoft.SharePoint.SPSecur...	983ff19b-0475-c0fa-03a6-fff304c21dba
01/03/2013 11:04:29.10*	OWSTIMER.EXE (0x0780)                   	0x1844	SharePoint Server             	General                       	aeyvf	Unexpected	...ity.RunWithElevatedPrivileges(WaitCallback secureCode, Object param)     at Microsoft.SharePoint.SPSecurity.RunWithElevatedPrivileges(CodeToRunElevated secureCode)     at Microsoft.Office.Server.UserProfiles.UserProfile.CreateSite(String strRequestUrl, Boolean bCollision, Int32 overrideCompatLevel, Int32 lcid)     at Microsoft.Office.Server.UserProfiles.UserProfile.CreatePersonalSiteWithSQM(String mySitePortalUrl, Int32 overrideCompatLevel, Int32 lcid)     at Microsoft.Office.Server.UserProfiles.UserProfile.CreatePersonalSite(Int32 lcid)     at Microsoft.Office.Server.UserProfiles.MySiteInstantiationWorkItemJobDefinition.ProcessWorkItemCore(SPWorkItemCollection workItems, SPWorkItem workItem, MySiteInstantiationWorkItemJobState timerJobState)     at Microsoft.Office.Server.UserProfiles.MyS...	983ff19b-0475-c0fa-03a6-fff304c21dba
01/03/2013 11:04:29.10*	OWSTIMER.EXE (0x0780)                   	0x1844	SharePoint Server             	General                       	aeyvf	Unexpected	...iteInstantiationWorkItemJobDefinition.<>c__DisplayClass1.<ProcessWorkItem>b__0(SPWorkItem wi, WorkItemTimerJobState tjs)     at Microsoft.Office.Server.Utilities.TimerJobUtility.<>c__DisplayClass1.<ProcessWorkItem>b__0()	983ff19b-0475-c0fa-03a6-fff304c21dba
```

The issue is that the entire method being called in the December 2012 hotfix for SharePoint 2013 is missing!  Here is the missing method as present in SharePoint 2013 RTM:

```csharp
        [SubsetCallableExcludeMember(SubsetCallableExcludeMemberType.PerSpec)]
        public Microsoft.SharePoint.SPSite SelfServiceCreateSite(string siteUrl, string title, string description, uint nLCID, int compatibilityLevel, int overrideCompatLevel, string webTemplate, string ownerLogin, string ownerName, string ownerEmail, string contactLogin, string contactName, string contactEmail, string quotaTemplate, SPSiteSubscription siteSubscription)
        {
            if (!this.WebApplication.SelfServiceSiteCreationEnabled)
            {
                throw new InvalidOperationException(SPResource.GetString(CultureInfo.CurrentCulture, "SscIsNotEnabled", new object[0]));
            }
            if (!string.IsNullOrEmpty(webTemplate))
            {
                SPWebTemplate template = this.GetWebTemplates(nLCID, overrideCompatLevel)[webTemplate];
                if (template.IsHideInSelfServiceCreation)
                {
                    throw new InvalidOperationException(SPResource.GetString(CultureInfo.CurrentCulture, "SscIsNotEnabledForTemplate", new object[0]));
                }
            }
            return this.WebApplication.Sites.Add(null, siteSubscription, siteUrl, title, description, nLCID, overrideCompatLevel, webTemplate, ownerLogin, ownerName, ownerEmail, contactLogin, contactName, contactEmail, quotaTemplate, this.Url, this.HostHeaderIsSiteName);
        }
```

This patch only breaks MySite self service site creation.  Standard self service site creation continues to function.  I do not recommend installing KB2752058 at this time.

References:

<http://www.toddklindt.com/blog/Regressions/SP2013Dec2012HotFix.aspx>

<http://www.toddklindt.com/SP2013December2012HotfixRegression>

Update: This is resolved by installing [KB2752001](http://support.microsoft.com/kb/2752001).  Validate that there is an appropriate Managed Path as specified by the Personal Site Location defined in the MySites Setup.  If the Managed Path is missing, an error similar to this will appear in the Application Event Log:

```text
Log Name:      Application
Source:        Microsoft-SharePoint Products-SharePoint Portal Server
Date:          1/24/2013 9:11:02 PM
Event ID:      5187
Task Category: Administration
Level:         Error
Keywords:      
User:          NAUPLIUS\s-sp2013farm
Computer:      SHAREPOINT2013.nauplius.local
Description:
My Site creation failure for user 'NAUPLIUS\trevor' for site url 'http://mysites2013/personal/trevor'. The exception was: Microsoft.Office.Server.UserProfiles.PersonalSiteCreateConfigurationException: A failure was encountered while attempting to create the site, wildcard inclusion used to create personal sites does not exist.
   at Microsoft.Office.Server.UserProfiles.UserProfile.<>c__DisplayClass2.<CreateSite>b__0()
   at Microsoft.SharePoint.SPSecurity.<>c__DisplayClass5.<RunWithElevatedPrivileges>b__3()
   at Microsoft.SharePoint.Utilities.SecurityContext.RunAsProcess(CodeToRunElevated secureCode)
   at Microsoft.SharePoint.SPSecurity.RunWithElevatedPrivileges(WaitCallback secureCode, Object param)
   at Microsoft.SharePoint.SPSecurity.RunWithElevatedPrivileges(CodeToRunElevated secureCode)
   at Microsoft.Office.Server.UserProfiles.UserProfile.CreateSite(String strRequestUrl, Boolean bCollision, Int32 overrideCompatLevel, Int32 lcid).
```

To add the Managed Path ("personal" in this case), run the following from the SharePoint Management Shell:

```powershell
New-SPManagedPath -RelativeURL personal -WebApplication http://mysites2013
```