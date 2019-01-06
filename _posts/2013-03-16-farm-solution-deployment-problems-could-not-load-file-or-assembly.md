---
layout: post
title: Farm Solution Deployment Problems – “Could not load file or assembly…”
tags: [development,sp2010,sp2013]
---

When building my most recent project, I had no issues deploying my solution from Visual Studio directly to the farm, however once the solution was packaged and deployed via PowerShell, I started running into errors...

```csharp
SharePoint : Failed to load receiver assembly "Nauplius.ADLDS.FBA, Version=1.0.0.0, Culture=neutral, PublicKeyToken=262c1486943f3872" for feature "FBA_STSHealthAnalyzer" (ID: d67ce2c7-8d09-4581-b0c4-3e1ce008911f).: System.IO.FileNotFoundException: Could not load file or assembly 'Nauplius.ADLDS.FBA, Version=1.0.0.0, Culture=neutral, PublicKeyToken=262c1486943f3872' or one of its dependencies. The system cannot find the file specified.
```

In the past, this has indicated the death of the project.  No amount of Timer Service restarts, iisresets, Process Explorer handle searches, or making sure the assembly\GAC_MSIL and assembly\temp folders are free of the binary would fix this.  Moving to a new project will usually resolve this, however that is a lot of work!

Instead, simply changing out the project's strong key assembly worked in this particular case.  If you haven't had to change this before, it is under the properties of the project -> Signing.  Under 'Choose a strong name key file', select New.  The password is optional.

After changing the strong name key file, the PublicKeyToken for the assembly will be updated.