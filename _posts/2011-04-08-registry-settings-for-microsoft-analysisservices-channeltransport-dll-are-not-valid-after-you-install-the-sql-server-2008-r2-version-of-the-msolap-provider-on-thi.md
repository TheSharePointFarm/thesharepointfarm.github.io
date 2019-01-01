---
layout: post
title: Registry settings for Microsoft.AnalysisServices.ChannelTransport.dll are not valid after you install the SQL Server 2008 R2 version of the MSOLAP provider on this computer
tags: [sp2010, powerpivot]
---

When installing PowerPivot in a multi-WFE scenario, you may need to manually register the `Microsoft.AnalysisServices.ChannelTransport.dll` on WFEs that do not have PowerPivot installed.

To do this, run a command prompt with an Administrator token.  Navigate to (and this path may change in the future, but hopefully not):

`C:\Windows\assembly\GAC_MSIL\Microsoft.AnalysisServices.ChannelTransport\10.0.0.0__89845dcd8080cc91\`

Once in that directory, run:

```text
C:\Windows\Microsoft.NET\Framework64\v2.0.50727\regasm.exe Microsoft.AnalysisServices.ChannelTransport.dll
```

Reanalyze the problem in Health Monitor and the issue will disappear.