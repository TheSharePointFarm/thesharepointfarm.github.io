---
layout: post
title: SQL Server 2012 PowerPivot Incorrectly Detects ADOMD.NET Causing a Failed Health Rule
tags: [powerpivot, sp2010]
---

When PowerPivot runs the health rule, [PowerPivot: ADOMD.NET is not installed on a standalone WFE that is configured for central admin](http://msdn.microsoft.com/en-us/library/hh230899.aspx), it checks for a file version of “11.0.0.0”:

```csharp
[SharePointPermission(SecurityAction.LinkDemand, ObjectModel=true)]
public override SPHealthCheckStatus Check()
{
	SPHealthCheckStatus passed;
	try
	{
		if (!HealthUtil.isCentralAdminConfiged())
		{
			passed = SPHealthCheckStatus.Passed;
		}
		else
		{
			try
			{
				if (HealthUtil.IsADOMDInstalled("11.0.0.0"))
				{
					return SPHealthCheckStatus.Passed;
				}
				passed = SPHealthCheckStatus.Failed;
			}
			catch (Exception exception)
			{
				UlsWriter.LogException(exception, UlsWriter.diagnostics.Administration);
				passed = SPHealthCheckStatus.Failed;
			}
		}
	}
	catch (Exception exception2)
	{
		UlsWriter.LogException(exception2, UlsWriter.diagnostics.Administration);
		throw;
	}
	return passed;
}
```

The file version is passed from this property in Microsoft.AnalysisServices.AdomdClient.AdomdConnection:

```csharp
public string ClientVersion
{
	get
	{
		if (this.clientVersion == null)
		{
			FileVersionInfo versionInfo = FileVersionInfo.GetVersionInfo(Assembly.GetExecutingAssembly().Location);
			this.clientVersion = versionInfo.FileVersion;
		}
	return this.clientVersion;
	}
}
```

The FileVersion property of the `Microsoft.AnalysisServices.AdomdClient.dll` (for SQL 2012) is:

`11.0.2100.60 ((SQL11_RTM).120210-1917 )`

Since that doesn’t compare to “11.0.0.0”, the rule fails.  This rule can be ignored for now, just make sure the proper `Microsoft.AnalysisServices.AdomdClient.dll` is installed at `C:\Windows\assembly\GAC_MSIL\Microsoft.AnalysisServices.AdomdClient\11.0.0.0__89845dcd8080cc91\`.