---
layout: post
title: August 2015 CU Error – “File names can’t contain the following characters”
tags: [sp2013]
---

Update October 2015: This issue is resolved in security update [KB3085582](https://support.microsoft.com/en-us/kb/3085582), SharePoint Foundation, SharePoint Server, and Project Server 2013 Cumulative Updates. Update to one of these patches rather than using the below method.

As noticed on a [TechNet forum post](https://social.technet.microsoft.com/Forums/office/en-US/5dc44e6e-b5bd-47f8-a75d-71c9841f9ace/august-2015-cu-list-attachments-give-file-names-cant-contain-the-following-characters-message?forum=sharepointadmin), the August 2015 CU for SharePoint 2013 has a new regression with attaching files to List Items. This can be reproduced with a Custom List (possibly others) by creating a new List Item, clicking "Attach File", identify any file that contains characters not listed in [KB905231](https://support.microsoft.com/en-us/kb/905231), then click OK. The error message "File names can't contain the following characters" will appear:

![FileNameError](/assets/images/2015/08/FileNameError.png)

A temporary workaround is available for this issue, but involves editing files in the 15 hive. Prior to performing this, make backup copies of these files. These backups must be in place prior to installing the next SharePoint patch.

There are two JavaScript files that must be edited:

`C:\Program Files\Common Files\microsoft shared\Web Server Extensions\15\TEMPLATE\LAYOUTS\FORM.debug.js`

`C:\Program Files\Common Files\microsoft shared\Web Server Extensions\15\TEMPLATE\LAYOUTS\FORM.JS`

For `Form.debug.js`, change the text starting on line 5307 from:

```js
    else {
        if (IndexOfIllegalCharInUrlLeafName(filename) != -1) {
            alert(Strings.STS.L_IllegalFileNameError);
            return;
        }
```

To:

```js
    else {
	var filNameOnly = filename.substring(filename.lastIndexOf('\\') + 1);
        if (IndexOfIllegalCharInUrlLeafName(filNameOnly) != -1) {
            alert(Strings.STS.L_IllegalFileNameError);
            return;
        }
```

And for `FORM.js`, find the following string:

``js
else{if(IndexOfIllegalCharInUrlLeafName(c)!=-1)
```

Change it to:

```js
else{var j=c.substring(c.lastIndexOf("\\")+1);if(IndexOfIllegalCharInUrlLeafName(j)!=-1)
```

Once completed, clear the browsers cache and re-test. If the error still exists, try using Inprivate browsing mode, which will not use the cache from the regular session.

A second workaround is also available, this does not involve any file editing.

In a List, create a List Item and save it without attempting to attach a file. On the List, highlight the specific List Item that was created. In the ribbon, click on Items -> Attach File. This dialog will allow you to successfully attach a file to a List Item.

A PSS case has been opened for this issue and the above solutions are the current official workarounds.