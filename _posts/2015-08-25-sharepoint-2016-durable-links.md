---
layout: post
title: SharePoint 2016 Durable Links
tags: [sp2016]
---

[This post is based on the SharePoint 2016 Preview — Functionality is not guaranteed to be supported, configured identically, or exist in RTM]

SharePoint 2016 [Durable Links](https://msdn.microsoft.com/en-us/library/mt346121(v=office.16).aspx) are the idea that a file can have it's name changed, or even move locations and the link will still function correctly. This way, if you send someone a link to a document, and down the road change the document name or location, the link stays valid. This feature requires a WOPI binding to Office Web Apps 2013 or presumably, the upcoming Office Online Server.

Durable Links can be captured via clicking on the ellipses of a document and directly below the document preview is a link in the textbox, or via a shared link in email.

As an example, I have a document at http://webUrl/subsite/Shared%20Documents/DurableTest.docx. In SharePoint 2013 and previous versions, if I changed the document file name or moved the document to http://webUrl/subsite/DurableDrop, the link would no longer be valid. With Office Web Apps configured, SharePoint 2016 generates Durable Links (in the format of http://webUrl/subsite/_layouts/15/WopiFrame.aspx?sourcedoc=%7B9CFFF9EE-FC73-4F16-992C-477382AD7710%7D&file=DurableTest.docx&action=default).

These WopiFrame URLs work just fine as Durable Links, but there is another [unsupported] option to make the links "pretty". Enabling pretty Durable Links consists of adding a Server Debug Flag to the farm:

```powershell
$farm = Get-SPFarm
$farm.ServerDebugFlags.Add(1028) #0x404 in hex :)
$farm.Update()
```

After enabling pretty Durable Links, we'll see that the file now has a query parameter to the document:

http://webUrl/subsite/Shared%20Documents/DurableTest.docx?d=w9cfff9eefc734f16992c477382ad7710

If I move the document to the DurableDrop library, it now becomes:

http://webUrl/subsite/DurableDrop/DurableTest.docx?d=w9cfff9eefc734f16992c477382ad7710

Or if I change the name:

http://webUrl/subsite/DurableDrop/ItsSureDurable.docx?d=w9cfff9eefc734f16992c477382ad7710

Or if I enter a random URL on the same Site Collection...

http://webUrl/myfile.docx?d=w9cfff9eefc734f16992c477382ad7710

The file continues to work with the original link of http://webUrl/subsite/Shared%20Documents/DurableTest.docx?d=w9cfff9eefc734f16992c477382ad7710!

There is a new property, LinkingUrl, that exists on the SPFile.

```powershell
$web = Get-SPWeb http://webUrl
$list = $web.Lists["Documents"]
$item = $list.Items[0]
$item.File.LinkingUrl
```

The parameter for the LinkingUrl value is simply a concationation of the full URL to the item, query parameter, plus the UniqueId of the file, `SPFile.UniqueId.ToString("N")`. To go full circle, the `LinkingUrl` is:

```powershell
$item.File.Web.Site.MakeFullUrl($item.File.Name) + "?d=w" + $item.File.UniqueId.ToString("N")
```

One caveat with this feature, unique permissions do not follow the document. If the user the item has been shared with does not have access to the Library the document is moved to, they will no longer have access to the document (although the link will continue to be valid).