---
layout: post
title: Workaround for April 2014 CU and MS14-022 Double Encoding Bug
tags: [sp2013]
---

9/9/2014 Update:

This issue is resolved in the [SharePoint 2013 September 2014 Cumulative Update](). Please use the CU instead of applying the below workaround.

-------------

The April 2014 Cumulative Update and MS14-022 introduced a "double encoding" issue that caused search results with %5C (the "\" character) to become double encoded (represented as "%255"). This caused broken links. This bug persists in the June 2014 Cumulative Update.

As a workaround, you can edit a few files in the 15 hive. This is a temporary workaround where the issue is a critical impact to your business.

You will want to edit two files, `Search.ClientControls.js` and `Search.ClientControls.debug.js`. Both files are located in `C:\Program Files\Common Files\microsoft shared\Web Server Extensions\15\TEMPLATE\LAYOUTS\` and I'd strongly suggest backing both files up to an alternate location so they can be restored prior to a final fix being released by Microsoft.

Edit `Search.ClientControls.debug.js` first. This file is used when debugging in the browser, so it is easier to read and will help you understand what needs to be edited in the non-debug version.

Find the following block of text:

```js
function $urlHtmlEncode(str) { // should be used only for href and src attributes
	if('string' === typeof str)
	{
		return encodeURI(Srch.U.ensureAllowedProtocol(STSHtmlDecode(str)));
	}
	else if(!Srch.U.n(str.value)) // we're doing this for backward compatibility 
	{
		return encodeURI(Srch.U.ensureAllowedProtocol(STSHtmlDecode(str.value)));
	}
	else // we're doing this to not to break any display template
	{
		return $htmlEncode(Srch.U.ensureAllowedProtocol(str)); 
	}
}
```

The encodeURI is a native function that is causing the double encoding issue. This is a fairly simple fix, just remove the encodeURI function and save the file. The block of text will look like this:

```js
function $urlHtmlEncode(str) { // should be used only for href and src attributes
	if('string' === typeof str)
	{
		return Srch.U.ensureAllowedProtocol(STSHtmlDecode(str));
	}
	else if(!Srch.U.n(str.value)) // we're doing this for backward compatibility 
	{
		return Srch.U.ensureAllowedProtocol(STSHtmlDecode(str.value));
	}
	else // we're doing this to not to break any display template
	{
		return $htmlEncode(Srch.U.ensureAllowedProtocol(str)); 
	}
}
```

The next step is to edit the non-debug version of the file. This one you'll want to be careful with as it isn't formatted as nicely. Search for "encodeURI" in `Search.ClientControls.js`:

```js
return encodeURI(Srch.U.ensureAllowedProtocol(STSHtmlDecode(a)));else if(!Srch.U.n(a.value))return encodeURI(Srch.U.ensureAllowedProtocol(STSHtmlDecode(a.value)));else
```

Next, remove encodeURI, carefully removing the opening parenthesis and closing parentheses for both method calls:

```js
return Srch.U.ensureAllowedProtocol(STSHtmlDecode(a));else if(!Srch.U.n(a.value))return Srch.U.ensureAllowedProtocol(STSHtmlDecode(a.value));else
```

Refresh your browser (you may need to clear your browser's cache), and the URLs, at least for People searches, will be correct. Other scenarios should be correct, but my lab is not set up to test them.