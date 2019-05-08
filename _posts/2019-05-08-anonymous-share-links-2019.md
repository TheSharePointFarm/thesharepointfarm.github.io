---
layout: post
title: Enabling Anonymous Share Links in SharePoint Server 2019
tags: [sp2019]
---

SharePoint Server 2019 does not support anonymous sharing links (aka guest links). However, it only takes four lines of PowerShell to implement. In addition, you get link expiration. This configuration does require Office Online Server as the guest sharing links leverage the web-based Office viewer/editor. Keep in mind, this isn't a supported configuration by Microsoft.

To enable this feature, run the following PowerShell from the SharePoint Management Shell:

```powershell
$farm = Get-SPFarm
$farm.Properties.Add("GuestSharingEnabled",$true)
$farm.Properties.Add("SPO-GuestSharingUIEnabled",$true)
$farm.Update()
```

When we visit a modern Document Library, we can now select the 'Anyone' option with link expiration:

![anon-share.png](/assets/images/2019/05/anon-share.png)

This yields a URL similar to the following:

```
https://devsp07.cobaltatom.com/sites/team/_layouts/15/guestaccess.aspx?guestaccesstoken=sn%2F%2FbPXsO9UB8ot8LpLUEoXYxawDnQsipOl34Q%2Fj8f8%3D&docid=1f71e8af92a384035b00ebcb2bc322f4a&rev=1&expiration=2019-05-08T07%3A00%3A00.000Z&e=OCqpdW
```

We can then see that a link is available with a set expiration:

![anon-link.png](/assets/images/2019/05/anon-link.png)

However you get the link to the end user (copy link or email), here is what the recipient sees (after enabling editing):

![anon-edit.png](/assets/images/2019/05/anon-edit.png)

Finally, when the link expires should you choose that option, this is what the recipient sees:

![anon-expire.png](/assets/images/2019/05/anon-expire.png)

And again, this is unsupported by Microsoft. After experimenting, you can disable this functionality by running the below cmdlets.

```powershell
$farm = Get-SPFarm
$farm.Properties.Remove()
$farm.Properties.Remove()
$farm.Update()
```