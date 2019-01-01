---
layout: post
title: Delete List link not removed from View All Site Content
tags: [sp2010]
---

Had a case with SharePoint 2010 where a user deleted a List (or Library) and the item was sent to the Site Collection Recycle Bin.  Once there, the link still remained present on the site's "View All Site Content".  When attempting to visit the link in View All Site Content, a 404 error would be generated.  Also, when attempting to create new content on the site (with a browser with Silverlight installed), a "List cannot be found" error message would appear.  Uninstalling Silverlight allowed the user to work around that issue.  I could also not restore the item from the Site Collection Recycle Bin.  Attempting to do so would produce an error of "List with this name already exists".

Finally, I deleted it from the Site Collection Recycle Bin as well as deleting it from the view of "Deleted items from end user Recycle Bin".  When deleting it from here, the link was immediately removed from the SharePoint site and everything worked properly again.