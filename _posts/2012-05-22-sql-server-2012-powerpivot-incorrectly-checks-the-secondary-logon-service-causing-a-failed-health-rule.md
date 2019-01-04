---
layout: post
title: SQL Server 2012 PowerPivot Incorrectly Checks the Secondary Logon Service Causing a Failed Health Rule
tags: [powerpivot,sp2010]
---

The Secondary Logon service is used for PowerPivot Workbook Thumbnail generation.  According to the PowerPivot 2012 Health rule reference (http://msdn.microsoft.com/en-us/library/hh230899.aspx),it is recommended to change the Secondary Logon service to a Startup Type of Manual, but the Health Rule checks if the Secondary Logon service has a Status of Running, not a Startup Type of Manual (or Automatic):

```csharp
[SharePointPermission(SecurityAction.LinkDemand, ObjectModel=true)]
public override SPHealthCheckStatus Check()
{
    SPHealthCheckStatus failed;
    try
    {
        ServiceController controller = new ServiceController {
            ServiceName = "seclogon"
        };
        if (ServiceControllerStatus.Running == controller.Status)
        {
            return SPHealthCheckStatus.Passed;
        }
        failed = SPHealthCheckStatus.Failed;
    }
    catch (Exception exception)
    {
        UlsWriter.LogException(exception, UlsWriter.diagnostics.Administration);
        throw;
    }
    return failed;
}
```

I’m unsure as to which is correct, the health rule itself or the health rule reference text, but to avoid this health rule from generating an error, make sure the Secondary Logon service is set to Automatic and Started on all SharePoint Servers in the farm.

This is not fixed in SQL 2012 Cumulative Update 1.