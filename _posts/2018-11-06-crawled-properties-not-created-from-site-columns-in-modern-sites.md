---
layout: post
title: Crawled Properties Not Created From Site Columns in Modern Sites
tags: [spo]
---

Crawled properties are not being created from Site Columns in Modern Sites when the column is properly implemented. This impacts SharePoint Online only; SharePoint Server 2019 is not impacted. These are the reproduction steps.

1. Create a Modern Team site
2. Create a new site Content Type and new Site Column added to the Content Type
3. Apply the Content Type to a List or Library
4. Add content to the List/Library and populate the custom column with information
5. Request a re-index of the List/Library or Site
6. Search for your crawled property after a period of time (you can wait for 24 hours just to confirm)

If you repeat the process for a classic site template, you should see your crawled property in the search schema in roughly 15 minutes or so. Remember, there is no SLA for search, so even on a classic site it may take quite a bit longer but the crawled property will eventually appear.

I contacted Microsoft Support on this issue and they confirmed that it was a known issue being worked on by the product group with no ETA on a resolution. Instead they provided me a workaround (but would not tell me why it works).

* Navigate to: https://tenantName.sharepoint.com/sites/siteCollectionName/_layouts/15/user.aspx
* Manually add your account to the site collection
* Request a re-index of the Site
* Wait for a period of time and the crawled property should present in the search schema, usually in a few minutes

This workaround does indeed work. I tested this and the crawled property (site column) appeared in the search schema within just a few minutes after requesting the re-index.