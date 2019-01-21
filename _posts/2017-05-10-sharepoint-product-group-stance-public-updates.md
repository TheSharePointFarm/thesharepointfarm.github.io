---
layout: post
title: SharePoint Product Group Stance on Public Updates
tags: [sp2016]
---

For quite a few years the advice many consultants and administrators would hand out regarding SharePoint updates (hotfixes, security fixes, and of course Cumulative Updates) was "wait and thoroughly test". Waiting was easy, but testing is always hard. In my personal experience, most companies do not have adequate test harnesses to cover the amount of basic testing that should occur in a production-like environment. This sometimes made CUs a "roll of the dice" for production deployments. And, in the past, certain fairly major issues with CUs have taken well over 4 months to be resolved; worse yet, these same issues would roll into moderate to critical security updates, preventing their deployment.

Now that we have SharePoint 2016, that stance has changed. The [patches are smaller](https://www.itunity.com/webinar/tech-talk-bill-baer-webisode-2-979), easier for Microsoft to test, and are released to a select group of individuals and companies prior to their public release. This allows Microsoft to receive reports and handle regressions prior to Patch Tuesday.

Because of these improvements, the SharePoint Product Group now recommends SharePoint 2016 Public Updates be deployed immediately upon release to the public. Now, I certainly won't put it against you to test the Public Updates in a production-like, but from personal experience, with regards to regressions, the 2016 Public Updates are significantly safer than any previous (or current!) SharePoint 2013 or lower patch level.

For an official document, see the article [Updated Product Servicing Policy for SharePoint Server 2016](https://technet.microsoft.com/EN-US/library/mt782882(v=office.16).aspx). The FAQ at the bottom of the article goes into the new stance on Public Updates.

>Question: Should I install the monthly Public Updates for SharePoint Server 2016 immediately or should I install them only if they contain a fix for a specific issue I'm having?

> Answer: Microsoft recommends that all customers install Public Updates for SharePoint Server 2016 as soon as they become available. Microsoft performs rigorous validation of each Public Update, both internally and with a select set of partners and customers before it is released to ensure it has the highest quality.