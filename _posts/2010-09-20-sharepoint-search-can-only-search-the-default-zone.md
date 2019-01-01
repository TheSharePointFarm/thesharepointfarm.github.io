---
layout: post
title: SharePoint Search can only Search the Default Zone!
tags: [sp2007]
---

SharePoint Search has some pretty silly limitations, one of them being that it can only search the “Default” zone.  If you've already created your site, set it up to be anonymous or forced SSL, SharePoint will be unable to crawl the site.

You may see events similar to this in the Application Event Log on the SharePoint server:

```text
Log Name: Application
Source: Windows SharePoint Services 3 Search
Date: 2/11/2009 6:50:06 PM
Event ID: 2436
Task Category: Gatherer
Level: Warning
Keywords: Classic
User: N/A
Computer: SPSERVER
Description:
The start address cannot be crawled.

Context: Application 'Search index file on the search server', Catalog 'Search'

Details:
The object was not found. (0x80041201)
```

To resolve this, you will have to extend your existing site to a new IIS web site, then modify Alternate Access Mappings to make the newly extended site in the Default Zone.

The first step is to go to the Application Management tab in the Central Administration.  Next, Create or extend Web application.  Click on Extend an existing Web application.

![image4](/assets/images/2010/09/image4.png)

First, choose your web application that you will be extending (in this case, we are extending example.com, which is a site with Anonymous access enabled).  Then, assign it a Host Header.  This Host Header should be either the name of the web server for simplicity's sake, or an internal-only accessible address.  Last, select the Custom zone (we are going to change this later on).

![image11](/assets/images/2010/09/image11.png)

After the site is extended, click on the Operations tab, and choose Alternate Access Mappings.

![image14](/assets/images/2010/09/image14.png)

Select Edit Public URLs.

![image17](/assets/images/2010/09/image17.png)

Change your Alternate Access Mapping Collection to the site you have extended:

![image23](/assets/images/2010/09/image23.png)

Next, simply flip-flop the Custom and Default URLs (you can also move your Internet-facing site to the more appropriate Zone named Internet)!

![image26](/assets/images/2010/09/image26.png)

Finally, to initiate a full crawl, open a command prompt on the SharePoint server, go to %ProgramFiles%\Common Files\Microsoft Share\dWeb Server Extensions\12\BIN and type:

```text
stsadm –o spsearch –action fullcrawlstart
```

After the crawl has finished, you should no longer see any errors related to Search in the Application Event Log.