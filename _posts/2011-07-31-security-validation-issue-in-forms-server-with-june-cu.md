---
layout: post
title: Security Validation issue in Forms Server with June CU
tags: [sp2010]
---

This is fixed in the August 2011 CU.  The method OnLoad no longer exists.

====

As noted at [Security Validation Issue – Form Services issue with SP1+June 2011 CU (Release 2)](http://ghamson.wordpress.com/2011/07/20/security-validation-issue-form-services-issue-with-sp1june-2011-cu-release-2-in-sp2010-sharepoint-msproject-projectserver/) and at [June CU Post Update Regression?](http://www.sharepointjoel.com/Lists/Posts/Post.aspx?ID=470), there is a security validation issue within Forms Server when switching views with a PeoplePicker or Attachment item within a Form.
The reason for this is the addition of an OnLoad event in the EntityEditor.
In SP1, the `EntityEditor.cs` in the `Microsoft.SharePoint.WebControls` namespace lacks an OnLoad event.  Unhelpfully, an OnLoad event exists in the June CU:

```csharp
[SharePointPermission(SecurityAction.Demand, ObjectModel=true)]
protected override void OnLoad(EventArgs e)
{
	if (this.Page.IsPostBack && !SPUtility.ValidateFormDigest())
	{
		throw new SPException(SPResource.GetString("PageSecurityValidationFailed", new object[0]));
	}
	base.OnLoad(e);
}
```

As noted in the above links, disabling Security Validation will work around this issue until Microsoft provides a fix.  Most likely what needs to happen is the ValidateFormDigest needs to occur both on initial page load as well as Post Back.

The Group check that fails as noted in the first link above happens with SP1 as well, so it is probably a non-issue (also as indicated that it happens with the attachment functionality of an InfoPath form as well as only on PostBack).