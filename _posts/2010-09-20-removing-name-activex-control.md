---
layout: post
title: Removing Name ActiveX Control
tags: [sp2007]
---

If you have a public-facing SharePoint site, you may want to remove the Name ActiveX Control from the site.  Users will otherwise get the Internet Explorer gold bar like this:

![image35](/assets/images/2010/09/image35.png)

Not very pretty, and most people will not want to mark your site as a Trusted site.  This will only occur on systems with Office 2003 or 2007 installed.

To remove the presence control entirely, open:

`C:\Program Files\Common Files\microsoft shared\Web Server Extensions\12\TEMPLATE\LAYOUTS\1033\INIT.JS`

Do a search for `ProcessImn`.  Once you find the below code, comment out both functions, so it reads like this:

```js
function ProcessImn()
{
//    if (EnsureIMNControl() && IMNControlObj.PresenceEnabled)
//    {
//        imnElems=document.getElementsByName("imnmark");
//        imnElemsCount=imnElems.length;
//        ProcessImnMarkers();
//    }
}
function ProcessImnMarkers()
{
//    for (i=0;i
//    {
//        if (imnCount==imnElemsCount)
//            return;
//        IMNRC(imnElems[imnCount].sip,imnElems[imnCount]);
//        imnCount++;
//    }
//    setTimeout("ProcessImnMarkers()",imnMarkerBatchDelay);
}
```

Save the file and your guests will no longer see the Name ActiveX Control gold bar.  If you do not want this to apply globally (e.g. if you’re running a farm for both internal and anonymous usage), follow the instructions at Microsoft’s KB931509.

Note the above change is immediate.  You do not need to restart SharePoint or IIS.