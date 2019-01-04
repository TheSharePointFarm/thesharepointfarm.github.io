---
layout: post
title: Reporting Services 2012 - “Some sites are not completely upgraded”
tags: [sp2010, ssrs]
---

If you’ve installed Reporting Services 2012 on SharePoint, you’ve probably noticed that the databases, like the below, are marked that some sites have not been completely upgraded:

Database | Type | Status
--- | --- | ---
ReportServer_Alerting | ReportingServiceAlertingDatabase | Database is up to date, but some sites are not completely upgraded.
ReportServerTempDB | ReportingServiceTempDatabase | Database is up to date, but some sites are not completely upgraded.

Of course, neither database contains “sites”, so there is nothing to upgrade.  If we look at the database properties using:

```powershell
$db = Get-SPDatabase | where {$_.Name -eq "ReportServer_Alerting"}
$db.NeedsUpgradeIncludeChildren
```

Returns true. If we look at the code, we enter the function:

```csharp
public bool NeedsUpgradeIncludeChildren
{
	get
	{
		return this.UpgradeManager.NeedsUpgrade(this, true);
	}
}
```

This function goes off onto a bunch of other functions, including functions that indicate that the `ReportingServiceAlertingDatabase` and `ReportingServiceTempDatabase` cannot be upgraded (or `CanUpgrade` is `false`).  We end up hitting this chunk of code:

```csharp
internal bool NeedsUpgrade(object o, bool bRecurse)
{
	bool flag;
	if (o == null)
	{
		throw new ArgumentNullException();
	}
	if (!Microsoft.SharePoint.Upgrade.SPUtility.IsMarkedUpgradable(o))
	{
		throw new ArgumentException(SPResource.GetString("MissingUpgradableAttribute", new object[0]));
	}
	try
	{
		flag = this.ReflexiveCanUpgrade(o);
	}
	catch (SPUpgradeException exception)
	{
		this.Log.Error(string.Format(CultureInfo.InvariantCulture, "ReflexiveCanUpgrade [{0}] failed.", new object[] { this.ObjectTitle(o) }), exception);
		flag = false;
		if (!bRecurse)
		{
			throw;
		}
	}
	if (!flag)
	{
		this.Log.Debug(string.Format(CultureInfo.InvariantCulture, "Cannot upgrade [{0}].", new object[] { this.ObjectTitle(o) }));
		this.Log.Debug(string.Format(CultureInfo.InvariantCulture, "Skip [{0}] NeedsUpgrade.", new object[] { this.ObjectTitle(o) }));
		return true;
	}
}
```

In the end, the value of flag is indeed false and we hit the two Log.Debug statements, which produce:

```text
08/11/2012 09:59:26.86     PowerShell.exe (0x17AC)                     0x1DCC    SharePoint Foundation             Upgrade                           fbv7    Medium      [powershell] [SPUpgradeSession] [DEBUG] [8/11/2012 9:59:26 AM]: Cannot upgrade [ReportingServiceAlertingDatabase Name=ReportServer_Alerting].     
08/11/2012 09:59:26.86     PowerShell.exe (0x17AC)                     0x1DCC    SharePoint Foundation             Upgrade                           fbv7    Medium      [powershell] [SPUpgradeSession] [DEBUG] [8/11/2012 9:59:26 AM]: Skip [ReportingServiceAlertingDatabase Name=ReportServer_Alerting] NeedsUpgrade.
```

And, because the statement returns true back to the original `NeedsUpgradeIncludeChildren` function, well, we get the Central Administration warning about some sites requiring an upgrade.  This is of course something to ignore.