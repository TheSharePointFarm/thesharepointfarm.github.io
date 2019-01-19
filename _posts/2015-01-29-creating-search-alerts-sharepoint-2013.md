---
layout: post
title: Creating Search Alerts in SharePoint 2013
tags: [sp2013]
---

Creating Search Alerts in SharePoint 2013 is a tad different than it was in SharePoint 2010. SharePoint 2010 had a SearchAlert class which allowed you to easily create the alerts. This class had a single dependency on the Web Application using it; there had to be a Search Service Application Proxy attached to the Web Application.

In SharePoint 2013, it is slightly different. The SearchAlert class constructor has been marked internal, and although you can access the constructor of the [SearchAlert class via reflection](https://shivabalachandran.wordpress.com/2013/01/30/lightening-up-the-world-on-creating-sharepoint-search-alerts-programmatically/), using reflection is unsupported.

The workaround is fairly simple, but it now has a dependency on the Web Application having a User Profile Service Application Proxy. Without the UPSA proxy, creating the alert will fail. Otherwise, this code is mostly gleamed from reflection.

```csharp
using (SPSite site = new SPSite(SPContext.Current.Site)
{
using (SPWeb web = site.OpenWeb())
{
var web = SPControl.GetContextWeb(this.Context);
var user = web.CurrentUser; //This is the user for which the alert will be created

KeywordQuery kQuery = null;
kQuery = new KeywordQuery(site);
kQuery.QueryText = "..." //Text to insert into the KeywordQuery, e.g. "(SharePoint")

SPAlertTemplate spAlertTemplate;
var spAlertTemplateCollection = new SPAlertTemplateCollection((SPWebService)site.WebApplication.Parent);

web.AllowUnsafeUpdates(true);

var spAlert = user.Alerts.Add();
spAlert.AlertType = SPAlertType.Custom; //Search Alerts are always Custom
spAlert.Title = "Title of Alert";
spAlert.User = user;
spAlert.User.Email = user.Email; //Or a string
spAlert.EventType = SPEventType.All;
spAlertTemplate = spAlertTemplateCollection["OSS.Search"] //This is the required template to use for Search Alerts!
spAlert.AlertTemplate = spAlertTemplate;
spAlert.MatchId = Guid.Empty; //This is required for Search Alerts and must be Guid.Empty
spAlert.Status = SPAlertStatus.On;
spAlert.DeliveryChannels = SPAlertDeliveryChannel.Email;
spAlert.AlertFrequency = SPAlertFrequency.Daily;
spAlert.Properties["p_query"] = string.Format("<Query Culture='en-US' EnableStemming='False' EnablePhonetic='False' EnableNickNames='False' IgnoreAllNoiseQuery='True' SummaryLength='185' MaxSnippetLength='92' KeywordInclusion='0' QueryText='{0}' QueryTemplate='' TrimDuplicates='True' Site='{1}' Web='{2}' KeywordType='True' HiddenConstraints=''/>", kQuery, site.Id, web.Id);
spAlert.Update(true)

web.AllowUnsafeUpdates(false);
web.Dispose();
}
}
```

The keys here are setting the AlertTemplate to "OSS.Search" and setting MatchId to Guid.Empty.