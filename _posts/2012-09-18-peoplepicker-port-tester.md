---
layout: post
title: PeoplePicker Port Tester
tags: [sp2007,sp2010,sp2013]
---

When it comes to the PeoplePicker and forests across trusts, a lot of individuals have difficulty with the port requirements.  Based on [Bill Baer’s](http://blogs.technet.com/b/wbaer/archive/2009/01/21/people-picker-port-protocol-requirements.aspx) excellent article, I’ve developed the [PeoplePicker Port Tester](http://peoplepicker.codeplex.com/) that will help with diagnosing what isn’t quite working when it comes to ports.  Simply run the application from your SharePoint WFE(s) to validate that they have the access they need to the remote forest.

Simply enter the path, filter used, as well as a username and password, then click Connect.

![PeoplePickerPortTester](/assets/images/2012/09/PeoplePickerPortTester.png)

If you have any suggestions, complaints, or ideas for this application, [let me know](http://peoplepicker.codeplex.com/discussions)!  Source code for this application can be found at the [CodePlex project page](http://peoplepicker.codeplex.com/SourceControl/list/changesets).