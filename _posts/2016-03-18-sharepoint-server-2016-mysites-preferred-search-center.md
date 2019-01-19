---
layout: post
title: SharePoint Server 2016 MySites Preferred Search Center
tags: [sp2016]
---

SharePoint Server 2016 MySites Preferred Search Center is the URL that MySites are supposed to take you to when you want to search across all content. Like previous versions of SharePoint, the User Profile Service Application has an option to set up the Preferred Search Center. Presumably, this is the URL that is used for searching the MySite. When you visit your MySite, there is a search box in the upper left hand corner, and when you start typing in it, you get this option to "Search Everything".

![OD4BSearchEverything](/assets/images/2016/03/OD4BSearchEverything.png)


You would think that the "Search Everything" would point to your Preferred Search Center as setup in the "Setup MySites" within the User Profile Search Application. It doesn't. Instead, it points to the Global Search Center URL as specified in the Search Service Application. Make sure the Global Search Center URL ends in /pages, otherwise the "Search Everything" feature will not function correctly.

![OD4BGlobalSearchURL](/assets/images/2016/03/OD4BGlobalSearchURL.png)

I'm unsure if this is a feature or a bug (I'm leaning towards bug), but just a point of warning that the behavior isn't as I would personally expect it to be.