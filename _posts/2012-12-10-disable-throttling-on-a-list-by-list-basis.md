---
layout: post
title: Disable Throttling on a List-by-List Basis
tags: [sp2010]
---

A Farm Administrator can disable Throttling on a list-by-list basis.  To do this, run the following:

```powershell
$web = Get-SPWeb http://webapp1
$list = $web.Lists["MyList"]
$list.EnableThrottling = $false
$list.Update()
```

To validate the Throttling settings are effective, run:

```powershell
$list.IsThrottled
```

If it returns `True`, the the List has been throttled (has more items than the List View Throttle limit).  If it returns false, it has either less than the List View Throttle limit or throttling has been disabled via the EnableThrottling property.

To re-enable throttling on a list, simply run

```powershell
$list.EnableThrottling = $true
$list.Update()
```

[Todd Klindt](http://toddklindt.com/) has a [PowerShell script](http://toddklindt.com/blog/Lists/Posts/Post.aspx?ID=283) to create a list with 5,001 items.