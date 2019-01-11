---
layout: post
title: User Profile Pictures, Exchange 2013, and SharePoint
tags: [sp2010,sp2013]
---

SharePoint can import user profile pictures from the [thumbnailPhoto](http://msdn.microsoft.com/en-us/library/windows/desktop/ms680034(v=vs.85).aspx) attribute in Active Directory as a JPEG.  This is done via setting up the Picture property within SharePoint's User Profile Application.  This is [well documented elsewhere](http://t.co/LZc8h0jHwo), so I won't go into the process.  However, if you set up a high quality image that fits under the thumbnailPhoto's 100Kb size limit, user profile pictures within SharePoint look excellent!

Now, what happens when we introduce Exchange 2013 and want to leverage it's new hi-res images for Exchange 2013 and Lync 2013?  Jeff Guillet, an Exchange Server MVP, has a [great article](http://www.expta.com/2012/12/working-with-hi-res-photos-in-exchange.html) on how to use the new Exchange 2013 cmdlet, [Set-UserPhoto](http://technet.microsoft.com/en-us/library/jj218694(v=exchg.150).aspx).  Set-UserPhoto does two things, it stores a copy of a very high resolution image (up to 648x648 px) in the Exchange 2013 user's mailbox.  The second thing it does is store a copy of the photo as a 48x48 px image in the thumbnailPhoto attribute.

So say SharePoint imports this 48x48 px image from the thumbnailPhoto attribute.  The thing you'll notice right away is that the image looks terrible.  It isn't the 48x48 px image itself that looks bad, but what SharePoint does to the image.  In the Microsoft.Office.Server.UserProfiles.UserProfilePhotos.CreateThumbnail internal method, SharePoint creates 3 different thumbnails for the imported photo of varying sizes:

Large = 300x300 px

Medium = 72x72 px

Small = 48x48 px

SharePoint is also using the highest quality options available to scale the image from the [System.Drawing.Drawing2D](http://msdn.microsoft.com/en-us/library/system.drawing.drawing2d.aspx) namespace:

```csharp
graphics.CompositingMode = CompositingMode.SourceCopy;
graphics.CompositingQuality = CompositingQuality.HighQuality;
graphics.InterpolationMode = InterpolationMode.HighQualityBicubic;
```

But, any way you slice it, a JPEG upscaled from 48x48 (thumbnailPhoto) to 300x300 px (user profile photo store) is going to look terrible.  So how do you get the best of both worlds?  My current suggestion would be to run this PowerShell, using both the Exchange [Set-UserPhoto](http://technet.microsoft.com/en-us/library/ee617215.aspx) cmdlet and the Active Directory Set-ADUser cmdlet.

```powershell
$photo = [byte[]](Get-Content C:\image.jpg -Encoding byte)
$photo2 = [byte[]](Get-Content C:\image100Kb.jpg -Encoding byte)
Set-UserPhoto -Identity jdoe -PictureData $photo -Confirm:$false
Set-UserPhoto -Identity jdoe -Save -Confirm:$false
Set-ADUser jdoe -Replace @{thumbnailPhoto=$photo2}
```

If anyone has a better suggestions, please let me know!