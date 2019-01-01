---
layout: post
title: After installing the August 2011 Cumulative Update, a health warning about User Profile Service jobs appears every hour
tags: [sp2010]
---

Every hour, a health error will appear regarding each User Profile Service Application instance regarding a missing Timer Job for the User Profile Service Application or User Profile Service Application Proxy:

![image3](/assets/images/2011/09/image3.png)

The ULS logs state:
```text
UserProfileInstalledJobsHealthRule.Check – missing job ‘Microsoft.Office.Server.UserProfiles.ADImport.UserProfileADImportJob’ for UserProfileApp ‘User Profile Services’
```

Microsoft included a new job that remains unused in the August 2011 Cumulative Update called the “UserProfileADImportJob”.
While Microsoft is checking for all job types, which includes this new job:

```csharp
public override SPHealthCheckStatus Check()
{
	SPHealthCheckStatus passed = SPHealthCheckStatus.Passed;
	if (UserProfileService.Service != null)
	{
		foreach (UserProfileApplication application in UserProfileService.Service.Applications)
		{
			foreach (Type type in UserProfileApplicationJob.Types)
			{
				if (!application.JobExists(type))
				{
					ULS.SendTraceTag(0x66326e6f, ULSCat.msoulscat_SPS_UserProfiles, ULSTraceLevel.Medium, "UserProfileInstalledJobsHealthRule.Check - missing job '{0}' for UserProfileApp '{1}'", new object[] { type, application.Name });
					passed = SPHealthCheckStatus.Failed;
				}
			}	
		}
	}
}
```

Microsoft does not install that job:

```csharp
internal void InstallJobs()
{
	this.EnsureAccess(base.Farm.TimerService.ProcessIdentity);
	foreach (Type type in UserProfileApplicationJob.Types)
	{
		if (type != typeof(UserProfileADImportJob))
		{
			this.InstallJob(type);
		}
	}
}
```

Perhaps the code was not ready for the August Cumulative Update?  Unsure, but you will be stuck with the Health error until the job is installed.  At least until the next CU, it should be safe to disable the missing job rule for the User Profile Application.

EDIT: This has been fixed in the October 2011 CU by ignoring the `UserProfileADImportJob` in the Health Check:

```csharp
public override SPHealthCheckStatus Check()
{
	SPHealthCheckStatus passed = SPHealthCheckStatus.Passed;
	if (UserProfileService.Service != null)
	{
		foreach (UserProfileApplication application in UserProfileService.Service.Applications)
		{
			foreach (Type type in UserProfileApplicationJob.Types)
			{
				if ((type != typeof(UserProfileADImportJob)) && !application.JobExists(type))
				{
					ULS.SendTraceTag(0x66326e6f, ULSCat.msoulscat_SPS_UserProfiles, ULSTraceLevel.Medium, "UserProfileInstalledJobsHealthRule.Check - missing job '{0}' for UserProfileApp '{1}'", new object[] { type, application.Name });
					passed = SPHealthCheckStatus.Failed;
				}
			}
		}
	}
}
```