---
layout: post
title: Using Microsoft Search Server (Express) to crawl an SSL Site
tags: [sp2007, mss]
---

Microsoft Search Server suffers from the same issue as SharePoint Search of not being able to crawl SSL-forced sites, however, it is able to crawl non-Default Zones.

In order to allow MSS(E) to crawl an SSL enabled site, extend the site to a new Zone in the SharePoint Central Administration.  You may enable SSL on the extended site, but you cannot force SSL on the site (which is done via IIS).

Once you've done this, configure the SSP for MSS(E).  On the Search Server, go to Crawl Rules.  Create a new Rule and use the following options as they appear in the image:

![image11](/assets/images/2010/09/image11.png)

Note that it is important to use an asterisk after the site URL (/*), otherwise Search Server will only crawl the root of the URL.  It is also important that “Do not allow Basic Authentication” is unchecked.

Next, if this extended site is not the Default Zone, go to Search Server Mappings and create a new mapping.  Enter the URL of the extended site under “Address in Index”, and enter the original site URL in the “Address in search results”.  This will transform the crawled URL into search results the users are able to access.

![image23](/assets/images/2010/09/image23.png)

The next time MSS(E) crawls the site, you should be able to get results from your searches.  You can also check the Crawl Log on the Search Server Administration site and verify there are no errors.