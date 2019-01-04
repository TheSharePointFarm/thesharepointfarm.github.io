---
layout: post
title: Property Section Name Cannot Start with “SPS-“
tags: [sp2010]
---

If you attempt to create a new User Profile Property Section with a Name of “**SPS**-_name_”, you’ll receive the following incorrect error message:

```text
Invalid Property name. Please enter a property name that matches the specification of URI schema name (RFC2396), which must begin with a letter, and must contain only letters, digits, and the characters ".", "+", or "-".
```

However, that error message isn’t accurate.  Instead, SharePoint reserves the use of “SPS-“ to itself.  Within Microsoft.Office.Server.UserProfiles.CoreProperty is a check to validate whether or not the new section starts with “SPS-“, and if so, throws an error:

```csharp
public override string Name
{
	get
	{
		return this.m_Name;
	}
	set
	{
		base.AdminCheck(StringResourceManager.GetString("PropertyUpdateAccessPermissionError"));
		if (!this.m_IsSection && !base.IsNewProperty)
		{
			throw new UpdateReadOnlyFieldException(StringResourceManager.GetString("Exception_Property_cs1"));
		}
		this.m_Name = Validation.SqlStringValidate("Name", value, 50, true, false, !this.m_IsSection);
		if (this.m_Name.StartsWith("SPS-", StringComparison.OrdinalIgnoreCase))
		{
			throw new InvalidValueException(StringResourceManager.GetString("Exception_Property_Name_SPSReservedPrefix"));
		}
		if (base.IsNewProperty)
		{
			this.m_OldName = this.m_Name;
		}
		base.m_IsChanged = true;
	}
}
```

![image7](/assets/images/2012/09/image7.png)

This incorrect error message occurs from at least as early as June 2011 v2 CU to the August 2012 CU. This incorrect error message also occurs when attempting to create a new Property with the name of "SPS-".