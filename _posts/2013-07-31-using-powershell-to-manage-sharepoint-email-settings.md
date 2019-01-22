---
layout: post
title: Using PowerShell to Manage SharePoint Email Settings
tags: [sp2010,sp2013]
---

Each SharePoint Document Library is capable of accepting Incoming Email.  Here is how to manage the settings via PowerShell.

In SharePoint 2010, the SharePoint Management Shell must run as the Application Pool account when setting the EmailAlias property.  Other properties can be set as a standard Shell Admin user.

First, bind to the Web and the List:

```powershell
$web = Get-SPWeb http://webUrl
$list = $web.GetList("LibraryTitle")
```

When making a change to a list property, make sure to call the Update() method, for example:

```powershell
$list.RootFolder.Properties.Add("vti_emailusesecurity", 1)
$list.RootFolder.Update()
$list.Update()
```

Here are the various settings you can apply.  I’ll be translating from the SharePoint UI to the PowerShell property.

![EmailLibrary](/assets/images/2013/07/EmailLibrary.jpg)

E-mail address (note this also sets Allow this document library to receive e-mail?)

```powershell
$list.EmailAlias = "string_value" #should be the email first part only, e.g. "test" in "test@example.com"
```

Group attachments in folders?

```powershell
$list.RootFolder.Properties.Add("vti_emailattachmentfolders", "string_value") #valid values are "root", "sender", and "subject"
```

Overwrite files with the same name?

```powershell
$list.RootFolder.Properties.Add("vti_emailoverwrite", 1) #1 = true, 0 = false
```

Save original e-mail?

```powershell
$list.RootFolder.Properties.Add("vti_emailsaveoriginal", 1) #1 = true, 0 = false
```

Save meeting invitations?

```powershell
$list.RootFolder.Properties.Add("vti_emailsavemeetings", 1) #1 = true, 0 = false
```

E-mail security policy:

```powershell
$list.RootFolder.Properties.Add("vti_emailusesecurity", 1) #1 = true - Use Document Library Permissions, 0 = false - Allow All/Anonymous
``

To disable Incoming Email on a Library, simply run:

```powershell
$list.EmailAlias = $null
$list.Update()
```