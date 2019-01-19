---
layout: post
title: How Initialize-SPResourceSecurity Works
tags: [sp2013]
---

`Initialize-SPResourceSecurity` or `stsadm -o -cmd secureresources` are responsible for security registry keys and the file system. But how does it work under the hood?

The command is fairly simple, it reads the registry keys under `HKLM\SOFTWARE\Microsoft\Shared Tools\Web Server Extensions\15.0\WSS\ResourcesToSecure\` and processes each value, applying the ACL to a registry key or on the file system.

Each key under `ResourcesToSecure` contains the following registry entries:

Permissions
ResourceName
ResourceType
SecurityGroup
SingleBoxOnly

The ResourceType has one of three values: RegKey (registry key), Directory, and File. This is the type of resource that the permissions will apply to. ResourceName is the path to the Resource Type.

Permissions have the following ACL mapping:

* R - Read
* W - Write (with implicit Delete, or Delete SubDirectories and Files)
* E - Execute
* D - Change Permissions (D? Really?)
* FC - Full Control

SecurityGroup is an bitmask (enum) and therefore can contain multiple security entries. The values are:

* 0 - WSS_WPG
* 1 - WSS_ADMIN_WPG
* 2 - LocalService
* 3 - NetworkService
* 4 - LocalSystem
* 5 - Users
* 6 - RESTRICTED (NT AUTHORITY\RESTRICTED)

The last value, SingleBoxOnly, will either be 0 or 1. It denotes if the SharePoint server was installed in a Single Server Farm role, but otherwise does not have an impact on securing resources.

Now you know how this cmdlet works!