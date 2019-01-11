---
layout: post
title: Microsoft, Save Us From Your Plugins!
tags: [sp2010,sp2013]
---

Plugins used to be one of the few ways to do anything interesting when integrating with the client.  We’ve gone a long way with HTML5, JavaScript, and CSS over the years, but are sometimes still straddled with ActiveX or NSAPI controls.  To give you an example, my company uses a helpdesk software that has roughly 5 ActiveX controls, and yes, Internet Explorer 6 optimized, just to view tickets!  SharePoint is no exception.  While SharePoint 2013 sheds some of our ActiveX/NSAPI (Firefox only) controls for this like the Datasheet View, they remain in other areas, such as the Name ActiveX controls or SharePoint 2010’s multiple upload control UI and so forth.

Google has [declared](http://blog.chromium.org/2013/09/saying-goodbye-to-our-old-friend-npapi.html) that Chrome will block NPAPI plugins in 2014 while Mozilla [will be blocking NPAPI](http://blog.mozilla.org/security/2013/01/29/putting-users-in-control-of-plugins/) plugins for Firefox in December 2013.  Not only is it time for the Internet Explorer PG to take a look at moving forward in this direction (which Modern IE has significantly helped, although there are a few ActiveX control exemptions), but the SharePoint PG as well.  Exchange (Outlook Web App) has rid itself of ActiveX controls, no longer providing different levels of support to different browsers (given they’re “modern”, supporting HTML5) and Office Web Apps also does not require any browser plugins.

Hopefully during the SharePoint 2013 cycle, Microsoft can look at eliminating [the rest of the SharePoint/Office ActiveX controls](http://technet.microsoft.com/en-us/library/cc263526.aspx#activex) in favor of light-weight cross-platform JavaScript solutions.