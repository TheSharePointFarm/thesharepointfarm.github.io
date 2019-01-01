---
layout: post
title: SharePoint Integrated Reporting Services “Jump to URL” Does Not Work
tags: [sp2007, ssrs]
---

If you use the “Jump to URL” feature within a Report, you will note that it works within the Visual Studio designer just fine, but once you deploy your report to a SharePoint Integrated Reporting Services instance, it no longer has a hyperlink associated with that field.  To solve that, use:

```vb
="javascript:void(window.open('" & "{URL Expression/Value}" & "'))"
```