---
layout: post
title: Incoming Email Service Job Lock Type Change between SharePoint 2010 and 2013
tags: [sp2010,sp2013]
---

EDIT: 12/12/2013: The LockJobType in SharePoint 2013 has been changed back to None in the December 2013 CU (<http://support.microsoft.com/kb/2837677/en-us>), allowing the SharePoint Administrator to run the Incoming Email Service on more than one server!

In SharePoint 2010, the Incoming Email service timer job had an `SPJobLockType` of `None`, which means that the job ran on all servers in the farm, given the Incoming Email service was provisioned.

In SharePoint 2013, that `SPJobLockType` has changed to `Job`, which means it only runs on a single member in the farm.  This is a problem for those who wish to use round-robin MX records.  While in SharePoint 2010, you could have multiple SharePoint servers, using equally weighted MX records servicing Incoming Email requests, in 2013, you can only have a single server servicing Incoming Email requests.  Disable the Incoming Email service on all SharePoint 2013 servers except the one designated by the MX record.  This will prevent the Incoming Email service timer job from running on a server which does not contain any email to be picked up.

SharePoint 2010:

```csharp
public SPIncomingEmailJobDefinition(string name, SPService service) : base(name, service, null, SPJobLockType.None)
{
}
```

SharePoint 2013:

```csharp
public SPIncomingEmailJobDefinition(string name, SPService service) : base(name, service, null, SPJobLockType.Job)
{
}
```

MSDN Reference: [SPJobLockType enumeration](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.administration.spjoblocktype.aspx)
