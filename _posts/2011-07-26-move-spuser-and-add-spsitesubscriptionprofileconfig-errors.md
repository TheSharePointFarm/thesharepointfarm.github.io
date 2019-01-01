---
layout: post
title: Move-SPUser and Add-SPSiteSubscriptionProfileConfig Errors
tags: [sp2010]
---

A final update to the Move-SPUser issue, and adding an Add-SPSiteSubscriptionProfileConfig issue as well.  With Move-SPUser, when running the process as a user other than the Farm Administrator, an error would occur with the value of the userProfileServiceApplicationProxy being null.  `Add-SPSiteSubscriptionProfileConfig` has a similar issue where the error is “Object reference not set to an instance of an object”.

The solution to both of these issues is pretty simple after all!  While of course you’ve probably added yourself to the Administrators of the User Profile Service Application in question, did you also give yourself permissions to invoke the service application?

If not, the solution is easy.  Go to Manage service applications.  Highlight your User Profile Service Application.  In the ribbon, under Sharing, click on the Permissions button.  Add yourself with Full Control.  You should at least see the Farm Administrator account in here as well.