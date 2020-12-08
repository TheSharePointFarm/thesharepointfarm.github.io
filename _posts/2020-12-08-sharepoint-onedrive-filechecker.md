---
layout: post
title: New and Updated FileChecker
tags: [spo,development]
---

[Bill Baer](https://twitter.com/williambaer) created a utility called [FileChecker](https://github.com/wbaer/FileChecker/). This tool parses a file server directory to call out rule violations for files you intend to migrate to OneDrive for Business or SharePoint Online.

I've forked the project and updated it in a few ways:

* Moved to .NET 5.0 (Core) as a console-only application
* Cross platform compatibility, including macOS and Linux
* Updated rules based off of the [current invalid file rules](https://support.microsoft.com/office/invalid-file-names-and-file-types-in-onedrive-and-sharepoint-64883a5d-228e-48f5-b3d2-eb39e07630fa)

You can find the new project at [FileChecker](https://github.com/tseward/FileChecker). I'm open to expanding this in any way. Binaries for Linux, macOS, and Windows are available in the releases section.
