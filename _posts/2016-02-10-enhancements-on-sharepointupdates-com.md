---
layout: post
title: Enhancements on SharePointUpdates.com
tags: [development,news]
---

I've added a few new enhancements on [SharePointUpdates.com](https://sharepointupdates.com). SharePoint Updates now has 481 entries as of February 2016 (technically 491, if you count the 'base' products) and covers FAST Search Server 2010, SharePoint 2010, 2013, Office Web Apps 2010, 2013, and as of a week ago, Workflow Manager 1.0!

There have been a couple usability and performance issues I've had to tackle with this dataset (even though it is small). The first performance issue came up once there were a couple of hundred records. Out of the box, DataTables is a client side solution -- that is, every time you visited the [Patches page](http://sharepointupdates.com/Patches), it would pull down every record. This made sorting and searching (or really, filtering) quite quick, but that initial load took way too long, especially on deprecated browsers like IE. So that was step one, to move the queries into code, but by doing so, we lost some of the native search functionality. What this update from today provides is a way to filter not only globally (the Search box), but also by product and/or patch type. This should help narrow down what you're looking for and make it much easier to use, especially for those who are using older products like SharePoint 2010 which do not bubble up to the top of the table.

![SharePointUpdatesFilters](/assets/images/2016/02/SharePointUpdatesFilters.png)

Hopefully this helps you discover the SharePoint patches for your platform. New patches are posted the same day Microsoft releases them. Any feedback is appreciated, let me know how it works for you!