---
layout: post
title: Error Setting the Regional Settings for a User Profile
tags: [sp2013]
---

EDIT: This issue is resolved in the May 2016 CU.

A SharePoint Administrator or user will receive an error setting the Regional Settings for a User Profile in the SharePoint 2013 February 2016 Cumulative Update. The error is specifically with the "Set Your Calendar" value.

![RegionalSettings](/assets/images/2016/03/RegionalSettings.png)

When either the user changes to "Always use my personal settings", as shown above, or the Administrator performs the same operation via the User Profile Service Application, SharePoint will generate an error due to an enumeration validation on the calendar type ([SPCalendarType](https://msdn.microsoft.com/en-us/library/office/microsoft.sharepoint.spcalendartype.aspx)). The February 2016 Cumulative Update, or more specifically, [KB3114719](https://support.microsoft.com/en-us/kb/3114719), includes this brand new validation method for the calendar type:

```csharp
private static object ValidatedCalendarType(object value)
{
    object obj2 = value;
    bool flag = false;
    Exception exception = null;
    try
    {
        short num = (short) value;
        if (Enum.IsDefined(typeof(SPCalendarType), num))
        {
            if (num == 0)
            {
                obj2 = (short) 1;
...
    catch (Exception exception2)
    {
        flag = true;
        exception = exception2;
    }
    if (flag)
    {
        string msg = StringResourceManager.GetString("Exception_UserProfileGlobal_InvalidCalendarType");
        ULS.SendTraceTag(0x310481, ULSCat.msoulscat_SPS_UserProfiles, ULSTraceLevel.High, "Error while trying to validate calendar type value. {0}. Exception: {1}.", new object[] { msg, exception });
        throw new PropertyInvalidFormatException(msg);
    }
...}
```

The issue is that the validation is performed using a C# [short](https://msdn.microsoft.com/en-us/library/ybs77ex4.aspx) (aka Int16), which is the type the variable "value" is (as well as "num"). The SPCalendarType [enum](https://msdn.microsoft.com/en-us/library/sbbt4032.aspx) is an [int](https://msdn.microsoft.com/en-us/library/5kzh1b5w.aspx) (aka Int32); without a cast, this prevents the validation from succeeding, regardless of the value passed to it as [Enum.IsDefined](https://msdn.microsoft.com/en-us/library/system.enum.isdefined%28v=vs.110%29.aspx) requires the object and enum target to be the same type. We can see this in the ULS logs.

```csharp
Error while trying to validate calendar type value. Invalid calendar type: The value must be in SPCalendarType enumeration. Note: "None" (Value=0) is not supported and will be automatically converted to "Gregorian" (Value=1).. Exception: System.ArgumentException: Enum underlying type and the object must be same type or object must be a String. Type passed in was 'System.Int16'; the enum underlying type was 'System.Int32'.    
 at System.RuntimeType.IsEnumDefined(Object value)    
 at Microsoft.Office.Server.UserProfiles.UserProfileGlobal.ValidatedCalendarType(Object value).
```

Whoops. This would be resolved by changing the line `System.Enum.IsDefined(typeof(SPCalendarType), (short) num)` to `System.Enum.IsDefined(typeof(SPCalendarType), (int) num)`. Simple fix. Unfortunately there is no work around at the moment as all methods pass through this validation (e.g. from PowerShell, SSOM, etc.).

A Microsoft PSS case has been opened.