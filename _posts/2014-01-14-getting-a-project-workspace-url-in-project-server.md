---
layout: post
title: Getting a Project Workspace URL in Project Server
tags: [pj2010,sp2010]
---

Project Server includes an optional Project Site for each Project. We can use the PSI, specifically WssInterop, to get the URL for the Project itself. The first thing you will need is the GUID of the project, which you can find in the address bar when viewing the project itself. The next steps are done in PowerShell:

```powershell
[Guid]$project = "GUID_OF_PROJECT"
$proxy = New-WebServiceProxy -Uri http://webAppUrl/PWA/_vti_bin/PSI/WssInterop.asmx?wsdl" -UseDefaultCredential
$proxy.ReadWssData($project).ProjWssInfo
```

The output will be similar to this:

```
WSS_SERVER_UID                  : 02aefa16-2628-4e31-a200-86340106c022
PROJECT_UID                     : 8f38b568-91bd-45c5-8563-f0485b0625ef
PROJECT_WORKSPACE_URL           : http://webapp1/PWA/Test1
PROJECT_ISSUES_URL              : http://webapp1/PWA/Test1/Lists/Issues/AllItems.aspx
PROJECT_RISKS_URL               : http://webapp1/PWA/Test1/Lists/Risks/AllItems.aspx
PROJECT_DOCUMENTS_URL           : http://webapp1/PWA/Test1/_layouts/viewlsts.aspx?BaseType=1
PROJECT_WORKSPACE_NAME          : PWA/Test1
PROJECT_WORKSPACE_VSERVER_URL   : http://webapp1
PROJECT_NAME                    : Test1
PROJECT_COMMITMENTS_URL         : http://webapp1/PWA/Test1/Lists/Deliverables/AllItems.aspx
WSS_PWA_ADMIN_ROLE_ID           : 1073741924
WSS_PWA_PROJECT_MANAGER_ROLE_ID : 1073741925
WSS_PWA_TEAM_MEMBER_ROLE_ID     : 1073741926
WSS_PWA_READER_ROLE_ID          : 1073741927
```

In the above example, http://webapp1 is the Web Application Url, PWA the Project Web Access instance, and Test1 is the name of the Project itself. And, as you can see, we have a SharePoint site created at http://webapp1/PWA/Test1 for the Test1 Project.