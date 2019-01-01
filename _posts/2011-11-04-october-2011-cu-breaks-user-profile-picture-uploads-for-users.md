---
layout: post
title: October 2011 CU Breaks User Profile Picture Uploads … For Users
tags: [sp2010]
---

Update 12/14/2011: This is resolved in the December 2011 Cumulative Update.

===
If an end user attempts to upload a picture to their own profile via their profile in My Sites, they will receive this error:

![image3](/assets/2011/11/image3.png)

There are some fairly significant code changes between the August 2011 CU version .5005 and the October 2011 CU.  For example, here is `LoadPictureLibraryInternal()` from `Microsoft.SharePoint.Portal.WebControls.ProfileImagePicture` from August:

```csharp
private void LoadPictureLibraryInternal()
{
	try
	{
		string importPhotoListName;
		if (this.m_profileType == ProfileType.User)
		{
			importPhotoListName = UserProfileGlobal.GetImportPhotoListName(this.m_profileType, this.m_objWeb.Locale);
		}
		else
		{
			importPhotoListName = StringResourceManager.GetStringWithCultureInfo(0x112d, this.m_objWeb.Locale);
		}
		foreach (SPList list in this.m_objWeb.Lists)
		{
			if ((list.BaseTemplate == SPListTemplateType.PictureLibrary) && (list.Title.Equals(importPhotoListName, StringComparison.OrdinalIgnoreCase) || (this.m_objPicLib == null)))
			{
				this.m_objPicLib = list;
			}
		}
	}
	catch (Exception exception)
	{
		ULS.SendTraceTag(0x37693038, ULSCat.msoulscat_SPS_UserProfiles, ULSTraceLevel.Medium, "ProfileImagePicture.LoadPictureLibrary: Could not load picture library: {0}.", new object[] { exception });
	}
}
```

And from October:

```csharp
private void LoadPictureLibraryInternal()
{
	try
	{
		this.m_objPicLib = UserProfileGlobal.LoadPictureLibrary(this.m_objWeb, this.m_profileType);
	}
	catch (Exception exception)
	{
	ULS.SendTraceTag(0x37693038, ULSCat.msoulscat_SPS_UserProfiles, ULSTraceLevel.Medium, "ProfileImagePicture.LoadPictureLibrary: Could not load picture library: {0}.", new object[] { exception });
	}
}
```

Microsoft made a change in what methods are called.  In the August CU, Microsoft calls (note “MyPage_MyList_ProfilePicture_Text” is a resource that maps to “User Photos”:

```csharp
internal static string GetImportPhotoListName(ProfileType profileType, CultureInfo ci)
{
	if (profileType != ProfileType.User)
	{
		throw new NotImplementedException();
	}
	return StringResourceManager.GetStringWithCultureInfo("MyPage_MyLists_ProfilePicture_Text", ci);
}
```

In the October CU, Microsoft calls:

```csharp
internal static SPList LoadPictureLibrary(SPWeb rootWeb, ProfileType profileType)
{
	SPList list = null;
	string importPhotoListUrl = GetImportPhotoListUrl(profileType);
	try
	{
		list = rootWeb.GetList(string.Format(CultureInfo.InvariantCulture, "{0}/{1}", new object[] { rootWeb.ServerRelativeUrl, importPhotoListUrl }));
	}
	catch (FileNotFoundException)
	{
	}
	if ((list != null) && (list.BaseTemplate == SPListTemplateType.PictureLibrary))
	{
		return list;
	}
	string format = StringResourceManager.GetString("Powershell_MovePictures_PictureLibNotFound_Text");
	throw new FileNotFoundException(string.Format(CultureInfo.CurrentCulture, format, new object[] { importPhotoListUrl }));
}
```

Which in turn calls (and which does not exist in the October CU):

```csharp
internal static string GetImportPhotoListUrl(ProfileType profileType)
{
	switch (profileType)
	{
		case ProfileType.User:
			return "User Photos";

		case ProfileType.Organization:
			return "Organization Logos";
	}
	throw new NotImplementedException();
}
```

So where is the problem?  The LoadPictureLibrary function above, specifically:

```csharp
{
	list = rootWeb.GetList(string.Format(CultureInfo.InvariantCulture, "{0}/{1}", new object[] { rootWeb.ServerRelativeUrl, importPhotoListUrl }));
}
```

”rootWeb.ServerRelativeUrl” = “/”.  “importPhotoListUrl” = “User Photos”.  So what happens during that string.Format?  We get the string “//User Photos”.  Is “//User Photos” a valid Uri?  Nope!  Hence, we fail.

Using a debugger, if we delete the extra “/” after being passed to into the below method, the we do not get the error:

```csharp
public SPList GetList(string strUrl)
{
	object obj2;
	if (string.IsNullOrEmpty(strUrl))
	{
		SPUtility.ThrowArgumentExceptionWithTraceTag(0x66386334, ULSCat.msoulscat_WSS_General, "strUrl", typeof(ArgumentNullException));
	}
	Guid empty = Guid.Empty;
	int itemId = 0;
	int typeOfObject = -1;
	string relUrl = Microsoft.SharePoint.WebPartPages.Utility.CanonicalizeFullOrRelativeUrl(strUrl, true);
}
```

Microsoft needs to modify the `LoadPictureLibrary` method to read:

```csharp
{
	list = rootWeb.GetList(string.Format(CultureInfo.InvariantCulture, "{0}{1}", new object[] { rootWeb.ServerRelativeUrl, importPhotoListUrl }));
}
```