---
layout: post
title: 
tags: [pj2010,sp2010,sp2013]
---

You may have a SharePoint installation where the servers have no Internet access. In this case, it is typically required that you run the prereqinstaller.exe with switches pointing to the binaries. But what if you didn't know where to get the binaries, such as WCF Data Services 5.6 now required by the SharePoint 2013 with Service Pack 1 installer? Simple! Use one of two programs: [Strings](http://technet.microsoft.com/en-us/sysinternals/bb897439) or [Process Explorer](http://technet.microsoft.com/en-us/sysinternals/bb896653).

Both of these processes can be done on any 64bit Windows computer. With Strings, simply mount the ISO or extract the contents to disk and run strings using the -u switch (Unicode) against the executable:

```powershell
strings.exe -u X:\prerequisiteinstaller.exe | findstr http://go
```

The result is:

![strings](/assets/images/2014/02/strings.png)

Alternatively, execute the `prerequisiteinstaller.exe`. On an unsupported Operating System (e.g., Windows 8) it will report that the tool does not support the current operating system. That's OK, just leave it running. Next, run Process Explorer "As Administrator" and find `prerequisiteinstaller.exe` in the Process list. Right click and go to Properties (or double-click), then select the Strings tab. Make sure the Image button is selected, and search for "http://go". All of the download links are grouped into this single section of the image.

![psexplorer](/assets/images/2014/02/psexplorer.png)

Alternatively, you can save the strings to text and search the text file.

Now you now how to find all of the pre-requisite download URLs!