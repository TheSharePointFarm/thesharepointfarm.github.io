---
layout: post
title: PowerPivot ASSPIInstallFarmAction Hang – Update
tags: [sp2010, powerpivot]
---

The [previous post]({% post_url 2011-03-11-powerpivot-asspiinstallfarmaction-hang %}) outlined a workaround for the hang caused by the ASSPIInstallFarmAction.  Instead of waiting for the hang, however, you can modify the `ConfigurationFile.ini` to point to the correct port.  When you are at the Ready to Install screen, notate the SharePoint Central Administration Port: .  Edit the file as specified in the Configuration file path below the installation summary.  Change the value of `FARMADMINPORT="nnnnn"` to the actual value of your Central Administration port.

![sqlppadminport](/assets/images/2011/03/sqlppadminport.png)

Note that if you click Back, and then Next again, it will re-write the ConfigurationFile.ini and you will need to modify it a second time.  After editing the ConfigurationFile.ini, just hit Install and it will pick up the port you have specified.

The ASSPIInstallFarmAction step still takes some time, so be patient, and follow the Detail.txt file in `C:\Program Files\Microsoft SQL Server\100\Setup Bootstrap\Log\<Date_Time>\`.