---
layout: default
title: Debugging SharePoint
---

SharePoint Debugging can be fun, as well as help you understand the platform.

There are a few ways to debug SharePoint (and always do it on a test installation, never production!):

1. Install Visual Studio on the problematic SharePoint Server, run locally
2. Install the Visual Studio Remote Debugger as a Windows Service

I prefer method #1 on a development server.  It ends up being less problematic in my experience than using a [Remote Debugger](http://msdn.microsoft.com/en-us/library/9y5b4b4f%28v=VS.90%29.aspx).

In order to do this, you must have [.NET Reflector VS Pro](http://www.reflector.net/vspro/) (or the trial).  This is essential as when debugging a process, it is aware of all of the libraries hooked to the process and can easily decompile them for you so you can step through the code.

If you are debugging against a Web Application, in the `web.config` for the Web Application, set to get more visibility when watching variables while debugging.

Finally, if you're needing to debug an Application Pool (say something in Central Administration is throwing an error, such as uploading pictures in the User Profile Service for users other than yourself in the June CU), open IIS Manager.  Find the offending Application Pool, and right click it and go to Advanced Settings.  Under Ping Enabled, set the value to False.  If the value is not set to false, the App Pool may recycle while you're attempting to debug it, causing your debugger to become disconnected (since the process shutdown).

The next step is to attach to the process that is giving the end user an error.  To do this, go to Debug -> Attach to process.  If you need to find out what w3wp.exe PID you should attach to, run `C:\Windows\System32\Inetsrv\appcmd list wp`.  That will output the Application Pool name and the corresponding PID.

In Visual Studio, go to the Debug menu and Attach to Process.  Select the offending process by PID. Click OK to any warnings.  Next, in the .NET Reflector menu and select Choose Assemblies to Debug.  .NET Reflector will read the assemblies loaded by the process.  When choosing which assemblies to debug, choose C# 3.0 for the version.  Pick the assembly(ies) in which you believe the error to be occurring.  It can be helpful to decompile associated assemblies as the code may jump (e.g. from `Microsoft.Office.Server.UserProfiles` to `Microsoft.SharePoint`) between assemblies.

If you are unable to determine where the error is exactly, when debugging, go to the Debug -> Exceptions menu.  Select one or more of the Common Language Runtime Exceptions.  Repeat the issue causing the error.

If you're looking to debug a Timer Job, attach to `OWSTIMER.exe` instead of one of the worker processes.

When performing an upgrade (hotfix, Cumulative Update, etc.), it is best to disable the Just In Time debugger.  To do this, use `regedit` and remove the following string:

```
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AeDebug
REG_SZ: Debugger
Value: "C:\Windows\system32\VSJitDebugger.exe" -p %ld -e %ld
```