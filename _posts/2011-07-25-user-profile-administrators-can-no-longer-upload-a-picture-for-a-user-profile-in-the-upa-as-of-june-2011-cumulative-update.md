---
layout: post
title: User Profile Administrators can no longer upload a picture for a User Profile in the UPA as of June 2011 Cumulative Update
tags: [sp2010]
---

Update: This is resolved in the December 2011 Cumulative Update.

====

There was a code change in the June 2011 Cumulative Update that prevents Administrators of a User Profile Application from uploading profile pictures for users other than themselves.
The code change occurred in `Microsoft.SharePoint.Portal.WebControls.ProfileImagePicker` function, within the `SaveFileToPersonalSite(HttpPostedFile file)` method.
Pre-June 2011 Cumulative Update:

```csharp
private void SaveFileToPersonalSite(HttpPostedFile file)
{
	SPUtility.ValidateFormDigest();
	if (Path.GetExtension(file.FileName).Equals(".gif"))
	{
		this.m_strErrorMsg = StringResourceManager.GetString(0x1e7f);
	}
	else
...}
```

June 2011 Cumulative Update:

```csharp
private void SaveFileToPersonalSite(HttpPostedFile file)
{
	SPUtility.ValidateFormDigest();
	if (!this.IsPersonCurrentUser())
	{
		this.m_strErrorMsg = Microsoft.SharePoint.Portal.WebControls.StringResourceManager.GetString(LocStringId.PictureUpload_SavePicture_Error);
	}
	else if (Path.GetExtension(file.FileName).Equals(".gif"))
...}
```

As you can see, in the June 2011 Cumulative Update, if the Current User is not the user being managed in the UPA, it will throw an error.  Unfortunately the error is not helpful (“There was an error saving the picture.  Please try again later”) to the Administrator, nor is anything useful logged in the Event Log.