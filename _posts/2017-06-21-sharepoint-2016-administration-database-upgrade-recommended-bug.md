---
layout: post
title: SharePoint 2016 Administration Database Upgrade Recommended Bug
tags: [sp2016]
---

If you've installed a new SharePoint 2016 farm and have noticed that the Administration database is in an Upgrade Recommended status, it may be a bug if you've applied any post-RTM, up to and including the June 2016 Public Update.

This issue appears on farms which have not activated Project Server via `Enable-ProjectServer`. Each SharePoint Content Database has a `dbo.Versions` table. In this table, each product (SharePoint, Project Server, among others) has one or more line entries for the version of schema that is applied to the database. Project Server has a `VersionId` of `A3DF1728-45F9-4E78-8177-FB5A83CD3225`. For a server without Project Server enabled, there is a single line entry with a `Version` of `4.0.0.0`. This appears to be the source of the problem. If this value is manually updated to the current schema version, or `16.1.284.0` in the case of the May 2017 PU, and the Administration database object invalidated, the database will be cleared of the upgrade recommended value. Of course _it is not supported to manually update SharePoint databases_ (nor is it proper for how SharePoint is tracking Project Server schema updates!), but the SharePoint Product Group is investigating this particular issue with the Administration database.