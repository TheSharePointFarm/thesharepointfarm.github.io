---
layout: post
title: 
tags: [sp2016]
---

As SharePoint Server 2016's codebase largely came from SharePoint Online, there are bits and pieces of interesting functionality that can be enabled on-premises. This is one of them, and is **not supported by Microsoft**! The purpose of this post is to hopefully ask you to [support such functionality on-premises](https://sharepoint.uservoice.com/forums/330318-sharepoint-administration/suggestions/13865781-enable-guest-links) and convince such functionality on-premises is important.

To enable the functionality, we only have three (or four) steps to get it working. The first step is that you must have Office Online Server integrated into the farm. Guest Links open up Office Online (for the appropriate file type). Second, we need to make some changes at the farm level.

```powershell
$farm = Get-SPFarm
$farm.Properties.Add("GuestSharingEnabled",$true)
$farm.Properties.Add("SPO-GuestSharingUIEnabled",$true)
$farm.Update()
```

The third step is to enable the functionality at the Site Collection level (note it cannot be restricted at any level below the Site Collection).

```powershell
$site = Get-SPSite https://sharepoint.nauplius.local
$site.ShareByLinkEnabled = $true
```

If the above does not return a value of true, the last step is to make sure you've disabled the RUM Global Provider Description and Simple Log Event Usage events in usage data collection.

Once you've done that, sharing your first item will create a list named 'Sharing Links' at the root of the Site Collection (so much like the Access Requests list, it is not under the /Lists/ path). This list holds data about the documents shared within the Site Collection.

When sharing a document with a Guest Link, the interface should be fairly familiar to those who have used SharePoint Online. You get two options to create a view only or an edit link.

![GuestSharing1](/assets/images/2016/11/GuestSharing1.png)

I've created both links for this example to show the view only and edit capabilities.

![GuestSharing2](/assets/images/2016/11/GuestSharing2.png)

When either one of these links is created, SharePoint will create a user (one user per link per document). As I've created two links here, one for view and the other for edit, we end up with two users for this document. The format of the users are 'SHAREPOINT\writer_<DocumentID>' and 'SHAREPOINT\reader_<DocumentID>' (note this is the internal ID of the document, not the Doc ID service).

```powershell
UserLogin                                          DisplayName
---------                                          -----------
SHAREPOINT\writer_782d1c8c44e5477fac52f97e7f85577f Guest Contributor
SHAREPOINT\reader_782d1c8c44e5477fac52f97e7f85577f Guest Reader
```

Because of this, anonymous access to the Site is not required -- we're using links that are assigned to authorized users! Subsequently, disabling the links will delete the users.

So onto our proof of concept. Does reader work?

![GuestSharing4](/assets/images/2016/11/GuestSharing4.png)

Yep! Notice the sign-in option as well as the lack of editing controls. And editing?

![GuestSharing5](/assets/images/2016/11/GuestSharing5.png)

Again, the option to sign in but with editing controls. And when the document is saved, it is updated by the appropriate user, too.

![GuestSharing6](/assets/images/2016/11/GuestSharing6.png)

Pretty neat! One thing to note is that this functionality does not work on the Modern UI for OneDrive for Business. While the option to create these types of guest links may be enabled, the API behind the scenes prevents the guest access links from being generated.

Again, if this functionality is of interest to you, I'd highly recommend upvoting or commenting on the [SharePoint UserVoice suggestion](https://sharepoint.uservoice.com/forums/330318-sharepoint-administration/suggestions/13865781-enable-guest-links). The more votes, the more likely it will get looked at! And a final warning, this functionality is not supported by Microsoft and this is simply for proof of concept purposes.