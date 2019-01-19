---
layout: post
title: Chrome Support of the SharePoint 2013 Media Webpart
tags: [sp2013]
---

As of this month, Chrome has disabled, by default, [NPAPI plugins](http://blog.chromium.org/2014/11/the-final-countdown-for-npapi.html). This means plugins, such as Silverlight and Flash, will no longer work by default. This include the SharePoint 2013 Silverlight-enabled Media Webpart.

Chrome does have a workaround to re-enable NPAPI plugins, which you can find on their [NPAPI deprecation developer guide](http://www.chromium.org/developers/npapi-deprecation). This can be done by navigating to chrome://flags/#enable-npapi, or via [Enterprise Policies](https://www.chromium.org/administrators/policy-list-3).

Support NPAPI plugins will be permanently removed as of September 2015; Silverlight and other NPAPI plugins will not function at all with no workaround available. It will be up to developers, such as Microsoft to support HTML5 standards and Chrome Extensions in place of NPAPI plugins.