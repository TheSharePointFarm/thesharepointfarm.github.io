---
layout: post
title: SQL Server Reporting Services ignores Alternate Access Mappings When Checking for Report Subscriptions
tags: [sp2010,ssrs]
---

For SQL Server Reporting Services 2008 R2 and 2012 in SharePoint Integrated Mode, SSRS is leveraging the exact URL the client is using in order to evaluate if a Report Subscription is present.  For example, I have a single Web Application with two AAMs: http://webapp1 and https://webapp1.nauplius.local.  Fairly common scenario.  This user (me) created a blank report and a dummy subscription on the report named Test1.  You can see the Subscription is present in the Manage Subscriptions list:

![Cap13](/assets/images/2012/10/Cap13.png)

However, we switch to the other AAM, and what do we see on the exact same report?

![Cap23](/assets/images/2012/10/Cap23.png)

No subscription!
In the `Microsoft.ReportServices.SharePoint.UI.ManageSubscriptionPages` method, Microsoft is creating the value for `m_reportUrl` using the current context of the URL the user is using and passing that to the `SubscriptionListControl`:

```csharp
protected override void CreateChildControls()
{
	if (this.InsideMainContent != null)
	{
		this.m_reportUrl = SPContext.Current.Web.Url + "/" + SPContext.Current.ListItem.Url;
	...
		this.m_subscriptions = new SubscriptionListControl(base.RSProxy, HtmlHelper.Current, this.m_reportUrl, this.m_deleteSubscriptions.Visible);
	...
	}
}
```

We go through a few other methods, but get down to `Microsoft.ReportingServices.SharePoint.SharedServices.Client.ReportingWebServiceClient.ListSubscriptions`:

```csharp
public ICollection<Subscription> ListSubscriptions(string itemPathOrSiteURL)
{
	return this.Execute<ICollection<Subscription>>("ListSubscriptions", proxy => proxy.ListSubscriptions(itemPathOrSiteURL));
}
```

We’re still passing down that `m_reportUrl` value from the page to this function, which finally passes it to:

```csharp
private TResult Execute<TResult>(string methodName, Func<IReportServiceManagement, TResult> action)
{
	TResult result = default(TResult);
	ReportingWebServiceApplicationProxy.Invoke<IReportServiceManagement>(this.m_serviceContext, methodName, this.m_siteUrl, this.m_xmlNamespace, delegate (IReportServiceManagement proxy) {
	result = action(proxy);
	});
	return result;
}
```

And because that `this.m_siteUrl` is the same value as `m_reportUrl`, if you are on a different Alternate Access Mapping than the Subscription was created on, we get no subscriptions.


