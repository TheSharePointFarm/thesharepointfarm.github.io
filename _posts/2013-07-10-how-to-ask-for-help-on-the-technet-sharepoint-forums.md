---
layout: post
title: How to Ask for Help on the TechNet SharePoint Forums
tags: [news]
---

I am currently a moderator on the [TechNet SharePoint](http://social.technet.microsoft.com/Forums/en-US/home?category=sharepoint) 2010 and 2013 General as well as Setup/Administration forums.  There are a lot of people asking for help every day, and I’m glad I’m apart of the community that helps not only manage, but to answer questions.

However, it helps those who are answering questions if you can provide some very basic information about what you’re attempting to accomplish or resolve.  Here are my suggestions for information that you should outline in your original post when asking for help:

* The build of SharePoint you’re using, which you can find by running `(Get-SPFarm).BuildVersion`
* If necessary, check the Control Panel “Programs and Features” to look at installed updates to validate Service Packs are installed
* The general architecture of the farm, number of SharePoint servers, their overall roles, number of SQL Servers; indicate if this is a single-server install, with or without SQL Express
* The error you’re seeing from an end user perspective (if applicable) and server perspective
* Validate you have checked all the log files for errors; the ULS, Application Event Log, System Event Log, and under Applications and Services, the SharePoint Event Logs
* Provide any information about ancillary servers and services, such as Exchange, AD RMS, ADFS or 3rd party SAML providers, and so forth
* Let us know your comfort level on accomplishing the task or resolving the error!  It helps target an answer at an appropriate level for the person asking the question

With all of this information, it helps those who are answering questions get off to a great start to quickly answer your question.  Many of us are aware, for example, of specific SharePoint Service Packs and Cumulative Updates that break specific features… knowing your farm build version helps answer this question without a lot of troubleshooting or back-and-forth.

One last suggestion would be to post the question in the correct forum.  I see a lot of developer questions in the General forums when they should have been asked in the Development forums.  Moderators can move your questions to the appropriate forum, but if one day goes by before the question is moved, it will likely be on the third or fourth page of the Developer forum and no one may see it.

If you have any other suggestions for information to provide up front when asking a question, I’d love to hear about it.