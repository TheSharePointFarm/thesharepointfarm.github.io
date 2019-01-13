---
layout: post
title: SharePoint Private Properties in PowerShell
tags: [sp2010,sp2013]
---

Welcome to the New Year! Hope everyone is having a great start!

Now back to SharePoint... Yet Another Originated From the MSDN/TechNet Forums Question (these are always fun). So there was a question about getting the databases that belonged to a specific User Profile Service Application. All of the UPA databases for the User Profile Service Application object are marked as Internal properties, in other words, not easily accessible.

So how can we get the databases that belong to a particular UPA? First, we need to know the properties that we're after, and for that I always use the handy .NET Reflector. The properties for the UPA Databases are:

* ProfileDatabase
* SocialDatabase
* SynchronizationDatabase

Next, get the named User Profile Service Application in PowerShell:

```powershell
$upa = Get-SPServiceApplication | where {$_.Name -eq "User Profile Service Application"}
```

Get the non-public Properties of the UPA object:

```powershell
$propData = $upa.GetType().GetProperties([System.Reflection.BindingFlags]::Instance -bor [System.Reflection.BindingFlags]::NonPublic)
```

And then the non-public Property that we're after:

```powershell
$socialProp = $propData | where {$_.Name -eq "SocialDatabase"}
$socialProp.GetValue($upa)
```

This will output the Social database Name, ID, and Type!

Here is the complete script:

```powershell
$upa = Get-SPServiceApplication | where {$_.Name -eq "User Profile Service Application"}

$propData = $upa.GetType().GetProperties([System.Reflection.BindingFlags]::Instance -bor [System.Reflection.BindingFlags]::NonPublic)

$socialProp = $propData | where {$_.Name -eq "SocialDatabase"}
$socialProp.GetValue($upa)

$profileProp = $propData | where {$_.Name -eq "ProfileDatabase"}
$profileProp.GetValue($upa)

$syncProp = $propData | where {$_.Name -eq "SynchronizationDatabase"}
$syncProp.GetValue($upa)
```

I hope that helps!