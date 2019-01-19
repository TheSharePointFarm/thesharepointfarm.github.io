---
layout: post
title: SharePoint DocKit 5.0.1 Review
tags: [review,sp2010,sp2013]
---

Full disclosure: MVPs are provided an SPDocKit Consultant Ultimate license, and I was also provided a Farm Ultimate license specifically for this review.

SPDocKit, at its core, is a documentation generation engine for SharePoint farms. It produces customizable documentation about the SharePoint farm that would take quite some time to otherwise script. Based on a [previous review]({% post_url 2013-06-01-sharepoint-dockit-3-2-1-review %}) of SPDocKit, this SharePoint DocKit 5.0.1 review will examine the new and revised features added since that time.

If you're upgrading from earlier versions of SPDocKit, like all updates for SPDocKit, the process is extremely easy. SPDocKit will detect the existing settings and upgrade everything as needed. Otherwise, there will be an option to create a new SQL database for SPDocKit. The SPDocKit database should on the SQL instance where the SharePoint Usage database is. This is a requirement in order to leverage functionality like the Feature Usage reports.

![SPDocKit5-1(/assets/images/2015/01/SPDocKit5-1.png)

Depending on the SPDocKit license, an option for enabling the SPDocKit Service will be available. This allows you to schedule snapshots of the farm configuration. The account running the SPDocKit service should have Local Administrator as well as Farm Administrator rights. In the case of this review, the Farm Administrator account will be used, but a dedicated account is likely more appropriate.

![SPDocKit5-2](/assets/images/2015/01/SPDocKit5-2.png)

Load Options provides choices for what type of data to collect about the farm, the more data that is collected, and the larger the farm, the longer this process will take to complete. This process can be very slow. If the SharePoint ULS and Windows Event Log options are selected, the Event Collection interval schedule will be available. This is where the snapshot schedule is available, depending on licensing. Once this configuration is complete SPDocKit will start running.

The first step is the Load Farm Settings. This process will load the settings selected, populate the database, and take a snapshot of the current state of the farm, ULS, and so forth. The first option is to load Farm Settings and Permissions. Choosing Permissions will likely add a significant amount of time to the data collection process.

![SPDocKit5-3](/assets/images/2015/01/SPDocKit5-3.png)

For the Farm Load, again, the more options selected, the longer the load time.

![SPDocKit5-4](/assets/images/2015/01/SPDocKit5-4.png)

Permissions Load allows the selection of the scope permissions should be loaded at. Loading down to the Item Level will incur a significant load time, even on a smaller SharePoint farm.

![SPDocKit5-5](/assets/images/2015/01/SPDocKit5-5.png)

While with the default settings for Farm Load, this test lab farm loads fairly quick as it has less than 1GB of content, selecting the option to load permissions at the Item Level causes very long load times. Think carefully before attempting to load the permissions at the List Item level.

![SPDocKit5-6](/assets/images/2015/01/SPDocKit5-6.png)

Once the selections are loaded, it will automatically close the window or indicate if an error occurred. If an error does occur, scroll up to the item marked Failed and click the link. There will be a link to the SPDocKit site with an explanation of why it may have failed along with possible resolutions.

The biggest functionality SPDocKit 5 brings to the table is Permissions Explorer. Not only can an administrator explore permissions down to the Item level, but it is also possible to manipulate permissions in a variety of ways. In this example, there are inherited permissions of the Announcement List. Only SharePoint groups are applied to the Annoucement List, yet it is possible to expand those groups to see who is a member of them.

![SPDocKit5-7](/assets/images/2015/01/SPDocKit5-7.png)

Each object (user or group) has a Properties page. This displays what type of account they are (AD User in this case) along with other basic information. Group membership is displayed of not only SharePoint groups within the Site Collection, but also Active Directory groups that may not be part of the Site Collection. This can help in identifying proper Active Directory groups to add to the site.

![SPDocKit5-8](/assets/images/2015/01/SPDocKit5-8.png)

The Permissions Explorer wizards are extremely powerful. From here it is possible to fully manage groups:

![SPDocKit5-9](/assets/images/2015/01/SPDocKit5-9.png)

And also manage memberships:

![SPDocKit5-10](/assets/images/2015/01/SPDocKit5-10.png)

![SPDocKit5-11](/assets/images/2015/01/SPDocKit5-11.png)

![SPDocKit5-12](/assets/images/2015/01/SPDocKit5-12.png)

Another welcome feature, which is not available in the SharePoint UI, is a breakdown of nested group memberships, as well as users who are apart of groups but are not in the User Information List. This can be handy as it is now possible to view permission levels based on Active Directory groups for an individual user who may not have yet visited the site.

![SPDocKit5-13](/assets/images/2015/01/SPDocKit5-13.png)

More great functionality is the Clone permissions wizard. The wizard lets an administrator duplicate permissions at the Web Application or Site Collection level. This is a great tool to have when onboarding new employees where domain-based groups are not in common usage.

![SPDocKit5-14](/assets/images/2015/01/SPDocKit5-14.png)

An administrator can also transfer permissions from one user to another; again another great feature when employees move on.

![SPDocKit5-15](/assets/images/2015/01/SPDocKit5-15.png)

Additional wizards include "cleaning" a Site Collection, where it is possible to remove users and groups en masse. The last two wizards involve managing the Site Collection Administrators group and SharePoint Permission Levels.

Queries and Rules is additional new functionality that is very powerful. Queries allow an administrator to gather and report on very specific information about Web Applications and Site Collections. For example, this custom built report on Site Collections and storage. From the report view, it is possible to filter and sort, just like Excel. And of course, the report can be exported to Excel, PDF, or Word.

![SPDocKit5-16](/assets/images/2015/01/SPDocKit5-16.png)

Within each Query is filtering functionality, as well as scheduling functionality. Reports are persisted, so it is possible to schedule an automatic query execution and review the results over time.

Rules allow an administrator to enforce specific behaviors on the target of the rule. In one of the samples provided, it will enforce Version History across Web Application(s), a Site Collection, Subsites of a Site Collection, or even specific Lists.

Jumping to Best Practices, and this was reviewed the last time around, again be careful with the recommendations here. There are bugs (such as the Distributed Cache Size recommendation exceeding the maximum recommendation of allocating 8GB, which should be fixed soon) and there are other recommendations that won't make sense for every environment. The rules follow "best practices", but not always "correct practices" for the particular environment.

One complaint from the previous review was the speed at which SPDocKit loaded the ULS and Event Logs. This issue has been rectified. Performance of log monitoring is significantly improved, and memory usage is significantly lower.

SPDocKit has expanded on the subscription functionality. It is now possible to output automated reports to file shares as well as directly into SharePoint Document Libraries. This is a welcome addition for those who do not want to be spammed with email, potentially on a daily basis. For the most part, everything that you see in the SPDocKit UI can be scheduled as an automated or manual report. From Farm Topology to Unique Permissions reports, the amount of data available from SPDocKit in an easy-to-consume fashion is extensive.

The generated documentation for the farm via DOCX or PDF is as great and customizable as ever. Even the default reports look professional and clean.

SPDocKit includes snapshot functionality. This is an automated process (although it can be taken manually) created by the SPDocKit Windows Service. Snapshot functionality isn't new, but it is something that needs to be mentioned as it is very valuable to see changes on a day-to-day basis with the farm. For example, this shows an increase in User Profile Count from one day to the next.

![SPDocKit5-17](/assets/images/2015/01/SPDocKit5-17.png)

Overall, this is one of my favorite 3rd party tools for SharePoint On-Premises. I'd highly recommend it to any administrator who does not want to manually document their farm. One thing I would like to see is the ability to run the desktop application in a fully interactive way from a client system, with a proxied connection to the SPDocKit Windows Service on a SharePoint server. This way it may be possible to have full functionality when running SPDocKit from a client, as currently the client relies on the SharePoint Server-side APIs, where the "work" is happening within the SPDocKit Windows Service, while the desktop application simply becomes a consumer of that data. In production environments, it is best practice to always manage remotely, but today that isn't possible with SPDocKit.

You can find SPDocKit at <http://www.spdockit.com>, and their Twitter account is [@SPDocKit](https://www.twitter.com/SPDocKit). With SPDocKit 5, Acceleratio introduced additional licensing tiers, which will allow you to pick the correct one for your specific needs. From personal experience, Acceleratio's customer support is very responsive and bugs are often fixed quite quickly. Because upgrades are painless, applying a bug fix just means installing over-the-top of the existing installation of SPDocKit.