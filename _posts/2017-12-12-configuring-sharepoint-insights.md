---
layout: post
title: Configuring SharePoint Insights
tags: [sp2016]
---

Have you tried configuring SharePoint Insights and run across this error?

![FailedToStopInsights](/assets/images/2017/12/FailedToStopInsights.png)

The solution for this is quite easy. Run a Command Prompt as administrator (cmd.exe or powershell.exe will do), navigate to the shortcut to where the Microsoft SharePoint Hybrid Configuration Wizard resides (your desktop), then run the following.

```powershell
".\Microsoft SharePoint Hybrid Configuration Wizard.appref-ms"
```

After that, going through the wizard allows you to successfully complete the SharePoint Insights configuration. As we’re calling the SharePoint OM, we need to run the Hybrid Configuration Wizard elevated in order for it to succeed.