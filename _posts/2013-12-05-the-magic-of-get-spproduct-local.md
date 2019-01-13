---
layout: post
title: The Magic of Get-SPProduct -Local
tags: [sp2010,sp2013]
---

Many of us have run into that situation where you've installed an update, started the Config Wizard, and you get a message that such-and-such server doesn't have these updates. But they're installed, because you just installed them, and SharePoint patches often won't let you reapply them as they "do not apply" to your system. So what do you do? Run `Get-SPProduct -Local` on the server "missing" the updates! And then you're able to run the Config Wizard. But why would a cmdlet with a Get verb fix something? Let's take a look...

First, Get-SPProduct enumerates the installed products on a SharePoint server. It does this by examining the registry key `HKLM\SOFTWARE\Microsoft\Shared Tools\Web Server Extensions\1x.0\WSS\InstalledProducts`. It looks at the `DisplayName` key for each product (product being represented by the key in the form of a GUID), and from there looks at the RequiredOnAllServers key, marking it as required if the key value is "1". Next, it takes the DisplayName value or registry key values from `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall`  and passes it through the Windows Installer service, calling [`MsiOpenProduct`](http://msdn.microsoft.com/en-us/library/aa370341(v=vs.85).aspx) and [`MsiGetProperty`](http://msdn.microsoft.com/en-us/library/aa370134(v=vs.85).aspx) to get the property of the `ProductName`, `ProductVersion`, and `RequiredOnAllServers` property.

Next, SharePoint continues looking at the MSI data, attempting to detect MSI patches using [`MsiGetPatchInfoEx`](http://msdn.microsoft.com/en-us/library/aa370128%28v=vs.85%29.aspx). From here, if it finds patches, it uses [`MsiOpenDatabase`](http://msdn.microsoft.com/en-us/library/aa370338(v=vs.85).aspx) to open the MSI database (this is why the Product Versions timer job fails if the Farm Administrator does not have Local Administrator rights) and runs a couple of queries. It also makes version comparisons between patches at this point. Finally, it places all detected products into a collection for use later.

The next step is to detect the upgrade status of the server, returning the status `UpgradeRequired`, `UpgradeAvailabile`, `UpgradeInProgress`, `InstallRequire`, `UpgradeBlocked`, or `NoActionRequired`.

Now we get into the part where the Get verb no longer makes sense. A T-SQL query is built using the `proc_RegisterProductVersion` stored procedure into one large SQL statement containing all detected products (named "BuiltProductsString" for reference below). For each product, this string contains the product name, GUID, Display Name, and if applicable, the patch GUID, KB article link, and the patch friendly display name. Once the string has been built, the process executes a stored procedure within the SharePoint Configuration database:

```sql
exec proc_ClearProductVersions '" + [Server_Guid] + "'\n" + [BuiltProductsString])
```

When the stored procedure completes, the result of the installed products is written out to the SharePoint Management Shell.

Another thing to note is that the process is the same for Get-SPProduct, the Product Version job timer job, joining a farm via Config Wizard or psconfig, and upgrading a SharePoint server via Config Wizard or psconfig.