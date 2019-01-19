---
layout: post
title: Context Sensitive List Search Broken by August 2015 CU
tags: [sp2013]
---

SharePoint Libraries include a search box for that specific library, e.g.:

![ListSearchBar](/assets/images/2015/12/ListSearchBar.png)

Prior to the August 2015 Cumulative Update, it was possible to find an item within the specific folder of a library. In the above example, if you went into Subfolder and then searched for an item, the search would be limited to items within that subfolder. After the August 2015 CU has been applied, the search box is no longer context sensitive. All items will be returned from the library that match the specific search term.

This is due to a change in the file INPLVIEW.js (and INPLVIEW.debug.js) where a function call was dropped which prevents the context sensitive search from working. As a temporary workaround, it is possible to modify these files to get this search functionality to it's original state.

The first thing that needs to be done is to make copies of INPLVIEW.js and INPLVIEW.debug.js, which are located in C:\Program Files\Common Files\microsoft shared\Web Server Extensions\15\TEMPLATE\LAYOUTS\. These copies must be restored prior to installing a SharePoint patch, otherwise the files may not be updated by a patch.

Next, in INPLVIEW.debug.js, open the file with a text editor and go to line 3058. This line will be within the function named "HandleRefreshViewInternal". Line 3058 should not have any text. Add the text:

```js
url = FixUrlFromClvp2(clvp, url, false);
```

For INPLVIEW.js, find the text `var b=c.ctx.queryString,a=c.ctx;`. Immediately after that text, add:

```js
b = FixUrlFromClvp2(a, b, false);
```

For INPLVIEW.debug.js, the entire function will look like:

```js
function HandleRefreshViewInternal(clvp) {
    var url = clvp.ctx.queryString;
    var ctxObj = clvp.ctx;
    url = FixUrlFromClvp2(clvp, url, false);
    if (typeof ctxObj.searchTerm != "undefined" && ctxObj.searchTerm != null)
        url = RemovePagingArgs(url);
    if (ctxObj == null || !ctxObj.IsClientRendering) {
        SubmitFormPost(url);
        return;
    }
    clvp.RefreshPaging(url);
}
```

And for INPLVIEW.js:

```js
HandleRefreshViewInternal(c){var b=c.ctx.queryString,a=c.ctx;b = FixUrlFromClvp2(a, b, false);if(typeof a.searchTerm!="undefined"&&a.searchTerm!=null)b=RemovePagingArgs(b);if(a==null||!a.IsClientRendering){SubmitFormPost(b);return}c.RefreshPaging(b)}
```

There is a PSS case open for this issue to revert the change back to pre-August 2015 CU behavior.