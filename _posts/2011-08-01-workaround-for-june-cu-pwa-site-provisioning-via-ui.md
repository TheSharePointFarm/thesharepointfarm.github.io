---
layout: post
title: Workaround for June CU PWA site provisioning via UI
tags: [sp2010, ps2010]
---

Update: This has been completely resolved in the December 2011 CU.

====

EDIT: The behavior has changed in the August 2011 CU.  After the second postback, the Administrator's name will become unresolved and will force the administrator to resolve the name.

If you attempt to configure a second (or more) Project Server Web Access site via the UI, you may encounter this error with SP1 and the June CU:

![image3](/assets/images/2011/08/image3.png)

A work around is to delete the Administrator name, and simply retype it.  For some reason, if you’re on the PWA creation page, and perform a postback (e.g. change Web Applications), there is an extra space prefixed to the Administrator name which causes the above error.  You can easily test this by selecting one Web Application, then another, and finally another; you’ll encounter the above error.