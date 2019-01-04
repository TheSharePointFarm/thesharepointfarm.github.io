---
layout: post
title: Incoming Email Distribution Group Approval
tags: [sp2010]
---

With Distribution Group approval, despite approving the Distribution Group in the Central Administration Distribution Groups List, I never saw the DGs created in Active Directory.  After watching the timer job a few times (named “job-pending-distribution-list”), it was apparent that the DGs were never leaving the “Pending” status.

In working with Microsoft, there is an undocumented “correct” way of approving DGs if DG approval is turned on in the Incoming Email settings.  Create your SharePoint Group and email-enable it as your normally would.  In Central Administration, the DG will be in the Pending approval state:

![image3](/assets/images/2012/11/image3.png)

One would think that simply highlighting the List item and hitting the Approve/Reject on the ribbon or selecting Approve/Reject from the drop down would function as expected.  But it doesn't.   Instead, highlight the List item and select View item.  On the View item pop up, there is a Distribution Approval:

![image7](/assets/images/2012/11/image7.png)

Click on Distribution Approval, and select Approved.  The DG will be created in Active Directory, and once the timer job named job-pending-distribution-list runs (by default, every 5 minutes), the status of “Creation request pending” on the SharePoint Group’s email address will disappear.