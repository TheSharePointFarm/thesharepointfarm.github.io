---
layout: post
title: BLOBCache Solution now available for SharePoint Server 2019
tags: [development,sp2019]
---

By popular demand (well, one person), the [BLOB cache solution](https://github.com/Nauplius/BLOBCache/releases/tag/2019-1.8) is ready for SharePoint Server 2019! The solution brings no new features as there aren't any new BLOB cache settings for SharePoint Server 2019.

If you're unfamiliar with this solution, it is a full trust solution that provides an interface in Central Admin -> Manage Web Applications to manage the BLOB Cache for a Web Application rather than having to edit the web.config by hand (never do this). This means that the settings persist in the Configuration database and deploy to your Web Application should you reprovision it or add a new SharePoint Server to the farm.

When you go to Manage Web Applications, a new BLOB cache button will be in the ribbon:

![bc-button](/assets/images/2019/05/bc-button.png)

You can then edit a variety of BLOB cache settings:

![bc-settings1](/assets/images/2019/05/bc-settings1.png)

Including flushing the BLOB cache and restoring the BLOB cache settings to their defaults (which will also disable BLOB cache since that's also a default).

![bc-settings2](/assets/images/2019/05/bc-settings2.png)

Make sure you plan on a mini-outage to make any BLOB cache settings change. As changing anything in the web.config by hand or through automated means recycles the IIS Application Pool in use by the IIS site, there will be a brief outage as you save these settings.

Enjoy!