---
layout: post
title: SharePoint Health Analyzer Rules Cannot be Automatically Activated During F5 Deployment
tags: [development,sp2010,sp2013]
---

Update: [KB2832238](http://support.microsoft.com/kb/2832238) was published which addresses the issue below.

When attempting to develop a SharePoint Health Analyzer rule and using F5 deployment, an error message will occur during the deployment process:

```text
Error occurred in deployment step 'Add Solution': Health analyzer rule registration 
requires that the {0} assembly be registered in the global assembly cache
```

This is because Visual Studio does not place the assemblies in the GAC, which is required for the Health Analyzer.

I opened a PSS case on this issue, and unfortunately this is a "won't fix" bug.  Here are the notes for CSS Case 113021510221063:

_The SharePoint 2010 Health Analyzer has a prerequisite that all assemblies registered for Health Analyzer rules have been installed in the Global Assembly Cache (GAC).  That requirement of GAC installation is present to ensure the reliability and stability of the Health Analyzer reports such that the SharePoint farm administrator can rely on Health Analyzer rules to reliably report on the health of their farm, and is enforced by the SPHealthAnalyzer.RegisterRules method.  During 'F5' debugging of a SharePoint solution, Visual Studio 2010 does not register the assemblies it builds in the GAC.  As a result the 'F5' debugging feature cannot be used for debugging a SharePoint solution which attempts to register a Health Analyzer rule in the assembly being debugged._

Note the above still holds true for Visual Studio 2012 with the Office Developer Tools.

The work around for this is to simply mark your Health Analyzer feature to not automatically activate on deployment, and manually activate it in Farm Features once deployed.