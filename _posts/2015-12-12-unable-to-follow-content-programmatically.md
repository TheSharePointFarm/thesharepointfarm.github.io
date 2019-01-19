---
layout: post
title: Unable to Follow Content Programmatically
tags: [sp2013]
---

[Developers have reported](https://social.technet.microsoft.com/Forums/sharepoint/en-US/68b9fc7e-7e91-4416-a7df-eae48a207460/programmaticly-follow-a-site-for-a-specific-user-is-not-working-anymore-after-installing-october) that after applying the October 2015 CU, they are unable to follow content programmatically using [SPSocialFollowingManager.Follow](https://msdn.microsoft.com/EN-US/library/office/microsoft.office.server.social.spsocialfollowingmanager.follow.aspx) method. While reported for the October CU, the change appears to be from the September 2015 CU, thanks to an [improved hybrid User Profile experience](https://support.microsoft.com/en-us/kb/3085481).

The September 2015 CU added the option to follow on-premises Sites and have the followed Sites appear in the user's SharePoint Online user profile. This is a great feature, but unfortunately broke the `SPSocialFollowingManager.Follow` functionality.

What has broken this functionality is `Microsoft.Office.Server.UserProfiles.FollowedContent.RefreshFollowedItem(FollowedItem item, SPS2SAppExecutionPolicy policy)`. This method now contains the line `item2.IsHybrid = this.Profile.IsFollowingHybridSetting() && (FollowedItemType.Site == item2.ItemType);`. The `IsFollowingHybridSetting()` method is calling `UserProfileServiceApplicationProxy.GetProxy(SPServiceContext.Current)`, however `SPServiceContext` is `null`. Because the service context is null, adding the followed item fails with the error:

```csharp
FollowedContent.RefreshFollowedItem(item.Url=http://spwebapp1/, policy=Allow) : Local execution failed; Could not refresh followed item http://spwebapp1/: Exception: Value cannot be null.  Parameter name: serviceContext
```

That said, there's a workaround! What needs to be done is to properly set `HttpContext.Current`. If, within the scope of your custom code, `HttpContext.Current` is set (even through a [console application](http://ddehaas.blogspot.com/2009/02/create-fake-httpcontext-for-sharepoint.html)), `SPServiceContext` will not be null and an item can be followed.