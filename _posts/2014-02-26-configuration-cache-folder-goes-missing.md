---
layout: post
title: When the Configuration Cache Folder Goes Missing…
tags: [sp2010,sp2013]
---

If the [SharePoint Configuration Cache]({% post_url 2014-02-07-what-is-the-sharepoint-configuration-cache %}) folder, along with the cache.ini file go missing, you'll likely notice the SharePoint Timer Service failing and restarting over and over again, along with a lack of scheduled timer jobs. The Timer Service failures will look like this in the System Event Log.

```text
Log Name:      System
Source:        Service Control Manager
Date:          2/26/2014 12:45:57 PM
Event ID:      7024
Task Category: None
Level:         Error
Keywords:      Classic
User:          N/A
Computer:      SPWFE01.nauplius.local
Description:
The SharePoint Timer Service service terminated with the following service-specific error: 
Unspecified error
```

Given this is the case, how do you restore the folder and cache.ini? The folder is a GUID, and unique to the farm, so creating just any folder isn't going to work.

The best method to recover the folder and file when no backups are available (and why would you backup a folder filled with transient  files?) is to take a backup of your Configuration database and restore it under a new name (this is done for supportability reasons, [SharePoint databases should be treated like black boxes](http://support.microsoft.com/kb/841057)). Then, query the Configuration database using the following command:

```sql
SELECT Id FROM Objects WHERE Properties like '%SPConfigurationDatabase%';
```

The returned value is the name of the folder that should reside in `C:\ProgramData\Microsoft\SharePoint\Config\`. Create this folder and create a new cache.ini. Within cache.ini, add "1" (without quotes) and save the file. Start the timer service (or as it is probably continuously crashing, it will start itself), and that should resolve the timer service crashes. Note that if the farm has multiple SharePoint Servers, follow the standard advice on other SharePoint servers that are working -- stop the SharePoint Timer service, and flush the cache, then after resolving the issue with the problematic SharePoint server, start the Timer service on all SharePoint servers.

Once the folder and cache.ini are recreated, and the Timer Service back up and running, the folder will be populated with XML files, the timer service will stop crashing, and finally timer jobs will start showing up as scheduled and running.