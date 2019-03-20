---
layout: post
title: A Flow Button for SharePoint Server 2019
tags: [sp2019]
---

Microsoft Flow is an cloud workflow/IFTTT-alike service. You can (and should!) use Flow with SharePoint Server 2019 as a replacement for many classic SharePoint 2010 or 2013 workflow features. But one thing is missing - there's no Flow button on Document Libraries or Lists, so I went ahead and created one.

![FlowExtension](/assets/images/2019/03/FlowExtension.PNG)

At the moment, this command extension is very simple. It _just_ supplies a button that drives the user to Flow, but sometimes a button is enough to get a user to think "what can I do with this?", and that alone is a success. If anyone has thoughts about further integration, you're absolutely welcome to contribute to the [source](https://github.com/Nauplius/SharePoint-2019-Flow-Extension).

Installation is simple: grab the current [release](https://github.com/Nauplius/SharePoint-2019-Flow-Extension/releases), upload it to your farm App Catalog, then add it like any other app to each site. No further configuration is required.

Note that the icon color for the button will not follow your site theme. Unfortunately command extension icons cannot be set dynamically in SPFx 1.41.