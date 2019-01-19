---
layout: post
title: PowerPivot Mid-Tier Process Account Does Not have Full Read
tags: [powerpivot,sp2010,sp2013]
---

The account running the IIS Application Pool leveraged by the PowerPivot System Service requires Full Read on any Web Application with the PowerPivot Service Application Proxy. However, the health rule governing validating this is true does not function correctly. The error in Review Problems and Solutions is:

MidTier process account should have 'Full Read' permission on all associated SPWebApplications

Let's dive in to why this is happening, and how to resolve it.

The first thing the Health Rule does is retrieve the process account name from the IIS Application Pool:

```csharp
GeminiServiceApplication application2 = application as GeminiServiceApplication;
GeminiServiceApplicationAccount item = new GeminiServiceApplicationAccount {
	serviceApplicationName = application2.Name,
	serviceAcctName = application2.ApplicationPool.ProcessAccount.LookupName().ToUpperInvariant()
```

The equivalent PowerShell is:

```powershell
$sa = Get-PowerPivotServiceApplication
$sa.ApplicationPool | Select ProcessAccountName

ProcessAccountName
------------------
NAUPLIUS\s-sp2013svcapppool
```

Once the Health Rule has the process account, the next step is to identify who has FullRead rights in the SPPolicy on the Web Application.

```csharp
foreach (SPWebApplication application in service.WebApplications)
{
	foreach (SPServiceApplicationProxy proxy in application.ServiceApplicationProxyGroup.Proxies)
	{
		flag = false;
		if ((proxy is GeminiServiceApplicationProxy) && string.Equals(proxy.Name, serviceApplicationName, StringComparison.OrdinalIgnoreCase))
		{
			foreach (SPPolicy policy in application.Policies)
			{
```

Now, here is where it fails.

```csharp
if (string.Equals(policy.UserName, b, StringComparison.OrdinalIgnoreCase))
{
	if (policy.PolicyRoleBindings.ToString().Contains(SR.FullReadPermission))
	{
		flag = true;
	}
	break;
```

What the above code is doing is validating that the strings are equal. It is comparing the UserName on the SPPolicy to the Process Account running is the IIS Application Pool. In a claims-enabled environment, this is never true as the claims identifier on the SPPolicy isn't equal to the Process Account.

As for a resolution, simply disable the rule and manually add the ProcessAccountName to the Web Application(s) with Full Read rights. This can be done via PowerShell:

```powershell
$wa = Get-SPWebApplication
$wa.GrantAccessToProcessIdentity("domain\username", [Microsoft.SharePoint.Administration.SPPolicyRoleType]::FullRead)
Disable-SPHealthAnalysisRule -Identity MidTierAcctReadPermissionRule -Confirm:$false
```

What about the automatic repair, and why doesn't that seem to work? Pretty simple, and the issue is along the same lines. What the automatic repair does it first look at a Web Application, then proceed to enumerate each User Policy on that Web Application. For each user, it adds Full Read rights (even though that user isn't the one running the IIS Application Pool the PowerPivot Service Application is assigned to).

```csharp
if (proxy is GeminiServiceApplicationProxy)
{
	foreach (SPPolicy policy in application.Policies)
	{
		string username = SPHttpUtility.NoEncode(policy.UserName);
		application.GrantAccessToProcessIdentity(username, SPPolicyRoleType.FullRead);
	}
```

The equivalent PowerShell code is:

```powershell
$wa = Get-SPWebApplication http://webAppUrl
$wa.GrantAccessToProcessIdentity("i:0#.w|nauplius\s-sp2013svcapppool", [Microsoft.SharePoint.Administration.SPPolicy]::FullRead)
Disable-SPHealthAnalysisRule -Identity MidTierAcctReadPermissionRule -Confirm:$false
```

When it does this, it grabs the username (say "i:0#w|domain\username") and calls `SPWebApplication.GrantAccessToProcessIdentity`. This method is unable to translate the claim value into a Windows Identity, thus we receive the exception "Some or all identity references could not be translated" and the the repair process exits.