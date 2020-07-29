---
layout: post
title: Stop Using Azure AD Security Groups in SharePoint Online
tags: [spo]
---

With SharePoint Server, it was strongly recommended to use Active Directory security groups to secure SharePoint resources, i.e. add users to the security group and nest the security group in the SharePoint group. This was primarily for search performance and query freshness for the newly added users. When assigning users to a SharePoint group, they may not see content until search has performed a security crawl on that content to update security scopes, and visa versa, a user may see content when they're removed from the object until a security crawl has completed.

If using Continuous Crawl, that guidance isn't recommended for on-premises either, however I have seen that practice continue to be carried forward on-prem and in SharePoint Online.

I strongly recommend against using Azure AD Security Groups outside of very specific scenarios for SharePoint Online and here's why:

* Search performance is Microsoft's domain
* Search architecture in SPO is vastly different from on-premises
* Using Azure AD Security Groups prevents end users from managing their own resources
  * And the iron fist of IT has made more than one SharePoint implementation underutilized or DOA
* You can't nest, as of this post, Azure AD Security Groups into Microsoft 365 Groups
  * This means access to certain resources, i.e. Microsoft Teams, has to be managed independently of Azure AD groups

The very specific scenarios I do recommend using Azure AD Security Groups are when you need dynamic groups to be added to SharePoint Online sites, i.e. you want a subset of users to have Visitor access to a SharePoint site and do not want to maintain group membership. For Team sites or Microsoft Teams, you can also do this with Microsoft 365 Groups which will prevent Owners from managing the Group membership, though they can manage who is an Owner.

Otherwise, let users get their work done -- it will reduce the workload for IT significantly with your help desk no longer needing to field permission requests. Even in an SMB sized org, this can mean you have a lot of time to do more important tasks.

If you need to add users to a number of sites at all once, look into [Azure Access Packages](https://docs.microsoft.com/azure/active-directory/governance/entitlement-management-access-package-first). This does require a P2 or EMS+E5 license for your users, but it has some other fancy functions as well, such as auto-removal of users after a time period or self-service via the Azure My Profile portal.

Think hard before using Azure AD Security groups for access management to Microsoft 365. There are often times better solutions available that allow end user flexibility to get their work done and free your time up for more important tasks.
