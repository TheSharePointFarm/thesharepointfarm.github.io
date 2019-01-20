---
layout: post
title: Customizing the Search Control Display Template
tags: [sp2013,sp2016]
---

I have customers who often want a custom search page of some sort, so I end up customizing the Search Control Display Template and making a minor query change. This most is more for my reference, but I figured these two simple things may help others.

When you create a custom page and drop search Web Parts (Search  Box, Search Result, perhaps Refinement), a user will navigate to the page and a bunch of default results show up. The fix for this is easy... Simply edit your Search Query.

![SearchBoxQueryDefault](/assets/images/2016/09/SearchBoxQueryDefault.png)

Replace `{searchboxquery}` with `\\{searchboxquery}`. No results will appear. EDIT: SharePoint MVP [Mikael Svenson](http://www.techmikael.com/) said it would be best to use `{?{searchboxquery}}` instead of `\\{searchboxquery}` -- do that instead! Note that the opening and closing brackets should surround the entire query, for example `{?{searchboxquery} Path:https://tenant.sharepoint.com/sites/projectSite}`.

![SearchBoxQueryFixed](/assets/images/2016/09/SearchBoxQueryFixed.png)

Unfortunately, you'll now see the "Nothing here matches your search" message when you navigate to the page. To fix that, using SharePoint Designer, navigate to the Search Display Templates under All Files -> _catalogs -> masterpage -> Display Templates -> Search. I often will make a copy of the Control_SearchResults.html file as I don't want to make changes to it to prevent impacting search results across the entire site. Give the copied file an appropriate name. Edit the file and change the value in the <title> attribute to distinguish this Display Template from the default Display Template (the title value is used as the display name in the Control Display Template drop down list).

Search for the text:

```javascript
if(ctx.ClientControl.get_shouldShowNoResultMessage()){
```

Simply change it to:

```javascript
var currentQueryTerm = ctx.DataProvider.get_currentQueryState().k; if(ctx.ClientControl.get_shouldShowNoResultMessage() && currentQueryTerm!=""){
```

Save the file. On SharePoint, edit the Search Results Web Part and change the Results Control Display Template to your new custom Control Display Template. When the user now navigates to the page, the 'no results' message is no longer displayed, until they search for a term that indeed has no results!

