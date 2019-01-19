---
layout: post
title: SharePoint Server 2016 Drag and Drop Control is Limited to 70Mbps
tags: [sp2016]
---

The SharePoint Server 2016 drag and drop control is limited to roughly an 8MB/s upload in the RTM and April builds when uploading a file in excess of 100 MB. Testing with files greater than 1GB will allow you to better see the transfer rate over a sustained period of time; I tested with Fedora-Live-Workstation-x86_64-23-10.iso which is 1,434,624 Kb, a file under the 2047 Mb limit allowed by SharePoint Server 2013 to make a direct comparison.

The test hardware here was identical for both VMs. Backend virtualized SQL Server is in use by both VMs. All VMs reside on the same Hyper-V host. Each VM has an equal amount of vCPUs (4) and vRAM (13 GB) allocated, along with the entire environment running on SSDs (SQL Server VM on a separate volume from the SharePoint VMs). Over CIFS, all three VMs, SharePoint Server 2013, SharePoint Server 2016, and the SQL Server transfer at near wire speed of 113 MB/s, or approximately 904 Mbps. The VMs and Hyper-V host all run on Windows Server 2012 R2.

These methods are not fully scientific, they're more or less rough estimates, but 'good enough' to show the performance disparity between the platforms.

Here is a transfer to SharePoint Server 2013 using the drag and drop control.

![SharePoint2013StandardDragDrop](/assets/images/2016/05/SharePoint2013StandardDragDrop.png)

The maximum sustained transfer rate was approximately 786 Mbps.

SharePoint Server 2016, out of the box.

![SharePoint2016StandardDragDrop](/assets/images/2016/05/SharePoint2016StandardDragDrop.png)

A quite odd see-saw effect, with a maximum transfer rate of approximately 73 Mbps. This upload takes forever.

And finally, SharePoint Server 2016 with an adjusted drag and drop control.

![SharePoint2016AdjustedDragDrop](/assets/images/2016/05/SharePoint2016AdjustedDragDrop.png)

Even better than SharePoint Server 2013, with a maximum sustained transfer rate of 979 Mbps! Nearly wire speed!

Because this requires an adjustment of a file in the 16 hive, I'm not going to recommend you make any changes to your farm implementation, but just be aware that large files in SharePoint Server 2016 may not upload as fast as you may have previously been used to with SharePoint Server 2013. For SharePoint Server 2016, this also impacts the standard 'Upload' control, if you choose to use that instead of the drag and drop method.

My _assumption_ for this 'limit' imposed on SharePoint Server 2016 comes from the [Core.LargeFileUpload](https://github.com/OfficeDev/PnP/tree/dev/Samples/Core.LargeFileUpload) PnP library. There is a note at the bottom of the readme.

> Internal testing has shown that a **slice size of 8 MB works best**, but of course this depends on the network connection you're using

This, of course, works out to about the performance we see. SharePoint Server 2016 uses the same StartUpload, ContineUpoad, and FinishUpload CSOM methods as outlined in that particular library (Method #3).

I don't know if the PG will make changes to the drag and drop control to improve performance for clients. I can certainly see why this makes sense in a SharePoint Online environment, but in a SharePoint on-premises environment, where network connectivity may equal or exceed 1 Gbps, it is a hindrance.