---
layout: post
title: TraceSeverity.None Throws Exception
tags: [sp2010,sp2013]
---

TraceSeverity flags are leveraged to report varying levels of criticality to the ULS log.  By properly leveraging the [TraceSeverity](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.administration.traceseverity.aspx) flag, a developer can provide critical information to the SharePoint Administrator to assist in diagnosing code or service-related issues.

As you can see, the TraceSeverity enumeration includes the flag of "None".  If a developer attempts to use TraceSeverity.None, it will throw an exception, similar to this:

```csharp
Unhandled Exception: System.ArgumentException: Specified value is not supported for the severity parameter. at Microsoft.SharePoint.Administration.SPDiagnosticsServiceBase.WriteTrace(UInt32 id, SPDiagnosticsCategory category, TraceSeverity severity, String output, Object[] data) at ConsoleApplication3.Program.Main(String[] args)
```

This appears to be by design.  In the SPDiagnosticServicesBase.WriteTrace method, we can see that Microsoft intended to throw this exception when using TraceSeverity.None:

```csharp
public void WriteTrace(uint id, SPDiagnosticsCategory category, TraceSeverity severity, string output, params object[] data)
{
    if (category == null)
    {
        throw new ArgumentNullException("category");
    }
    if (!Enum.IsDefined(typeof(TraceSeverity), (int) severity) || (severity == TraceSeverity.None))
    {
        throw new ArgumentException(string.Format(CultureInfo.InvariantCulture, SPResource.GetString("InvalidArgumentText", new object[0]), new object[] { "severity" }));
    }
```

In addition, using [EventSeverity.None](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.administration.eventseverity.aspx) will throw a similar error.