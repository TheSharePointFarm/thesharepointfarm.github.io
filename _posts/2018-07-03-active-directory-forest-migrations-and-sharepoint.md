---
layout: post
title: Active Directory Forest Migrations and SharePoint
tags: [sp2016]
---

Active Directory forest migrations are an uncommon but sometimes necessary task. There are many routes to perform a successful migration, but SharePoint has special considerations when doing so. This information applies to SharePoint 2010 with the August 2010 Cumulative Update or higher (if you're below that patch level, it's time to patch current!).

I will be making some assumptions:

* Two-way forest full trust is in place
* SID History is used for the migration
* The target farm and it's associated service accounts will not be migrated and/or are already in the target domain
* You are using ADMT or another domain migration tool such as Migration Manager for Active Directory to migrate user accounts and Active Directory groups

In this scenario, I will be referring to lab.cobaltatom.com as the source forest and xenonatom.com as the target forest. With this exercise, I will be going over important notes for consideration that are in-scope for a SharePoint farm but may not be in scope for an Active Directory administrator.

It is extremely important to note that, when using SID History ([sIDHistory attribute](https://docs.microsoft.com/en-us/windows/desktop/ADSchema/a-sidhistory)) that allowing two accounts to have the same SID in a trust scenario defeats the Microsoft security model -- that is, each user has a unique SID. SID History does play into this as SID History is evaluated along side the user's SID ([objectSid attribute](https://docs.microsoft.com/en-us/windows/desktop/ADSchema/a-objectsid)).

Even prior to migrating any objects, generally you will want to set up the People Picker (2013+, not necessary with 2010 for a two-way trust) to support the trusted forest. This can be done via [PowerShell]({% post_url 2014-01-15-powershell-for-people-picker-properties %}) and does not need an Application Key nor user account set on the connection as the forest is fully trusted; make sure that both the source and target forest are listed in the `SearchActiveDirectoryDomains` object. The next step is to set the farm property `HideInactiveProfiles`. This can be done with PowerShell:

```powershell
$farm = Get-SPFarm
$farm.Properties('hideinactiveprofiles','true')
$farm.Update()
```

What this property does is it tells SharePoint to ignore disabled Active Directory accounts. This is especially important in farms where the service accounts for SharePoint reside in the target forest. SharePoint uses a built-in Windows method [LsaLookupNames2](https://docs.microsoft.com/en-us/windows/desktop/api/ntsecapi/nf-ntsecapi-lsalookupnames2) to resolve usernames. This method will attempt to resolve usernames in the local domain prior to searching domain/forest trusts. We do not want SharePoint to find our inactive target accounts. Note this property cannot be used in a one-way trust scenario.

The second property we need to set is to ignore disabled user accounts in the People Picker. We can again us PowerShell to do this on each Web Application.

```powershell
foreach($wa in Get-SPWebApplication) {
    $wa.PeoplePickerSettings.ActiveDirectoryCustomFilter = '(!userAccountControl:1.2.840.113556.1.4.803:=2)'
    $wa.Update()
}
```

This will prevent users from finding disabled accounts when querying via the People Picker.

The next step is to set up your User Profile Service synchronization connections. Using AD Import and UPSS are straight forward -- you'll want to prevent the import of disabled accounts (remember, AD Import does not filter out profiles that have been subsequently disabled post-import; User Profile Sync Service does remove those profiles). MIM is not so straight forward. As there is no longer any 'magic' happening in SharePoint's code to handle profile joins, you will need to handle those within MIM on your own. My article on [SharePoint, Microsoft Identity Manager, and Forest Migrations](2018-03-15-sharepoint-microsoft-identity-manager-and-forest-migrations) can help you set up the join process in MIM.

This is where it is critical that only a single object be active at any one time between the forests. If both objects are active, the User Profile service may migrate the users back and forth within SharePoint as the synchronization process runs.

The last step that you may need to perform is to increase the value of [LsaLookupMaxCacheSize](https://support.microsoft.com/en-us/help/946358/the-lsalookupsids-function-may-return-the-old-user-name-instead-of-the) on your SharePoint servers. This is a Windows value that out of the box maps 128 usernames to SIDs in a local cache for performance purposes. It prevents Windows from having to query the DC for an object already in this cache. However, it may be possible that Windows has such a large rate of cache evictions that it identifies the disabled account in the target forest as the active identity rather than the active source account. This behavior will show up in a couple of ways, such as IIS logs reflecting the disabled account in the target domain, or the ULS logs showing the correct source object UPN a disabled sAMAccountName in the target forest. This will also cause User Profiles that are unattached from their domain object (which also means generating a new MySite). The linked support article goes through the process of changing the value. If you have a fairly small domain (just a couple of thousand user accounts that interact with SharePoint), you can consider setting it to a value higher than the number of accounts. Each cached object will take a couple of kilobytes of memory. If you have a very large domain, set the value high but monitor memory usage of the SharePoint servers). There is one thing to keep in mind; the cache is evicted by default every 7 days for any particular entry, however if a sAMAccountName changes while the previous sAMAccountName to SID mapping is in the LsaLookup Cache, the user will fail to authenticate to the server (in other words, they'll get an access denied error). The only way to flush this cache is to reboot the server. If your company has a policy of changing usernames when requested, consider placing that policy on hold until the forest migration is completed. The LsaLookupMaxCacheSize value can be deleted or set to 128 post-migration to return to the default value. Reboot the server for the reduction in cache size to take effect.

At this point in time, your Domain Admins will probably have performed a sync of objects from the source to target forest, disabling the target objects; this is common practice in a forest migration to stage the objects. This would include synchronizing Active Directory [security] groups and their membership.

With Active Directory Groups, you should add the duplicate group name from the target forest to the ACL within SharePoint. We cannot use the source Active Directory security group with a user in the target forest as SharePoint does not understand [Foreign Security Principals](https://docs.microsoft.com/en-us/windows/desktop/ADSchema/c-foreignsecurityprincipal), which is what is created when you add a user to a group in a cross forest scenario. These objects are 'pointers' and not complete presentations of a user object in Active Directory. There are tools out there to assist with duplicating object security, such as Sharegate, Metalogix ControlPoint, SPDocKit, among others.

Prior to performing the internal SharePoint migrations, make sure you have Full Control under Administrators and Permissions on the User Profile Service Application. If you make the permission change while you have the SharePoint Management Shell open, close it prior to performing a migration otherwise you'll receive an error.

The last step is when all the hard work occurs. The Domain Admin starts performing the user migration. Using ADMT or another migration tool of choice, the administrator will disable the source forest account and enable the target forest account. At this point in time, the SharePoint Admin must perform a migration within SharePoint, as well. This can be done through PowerShell, but it is more convenient to do through stsadm.

```powershell
$user = Get-SPUser -Identity 'i:0#.w|lab\jdoe' -Web https://siteCollectionUrl
Move-SPUser -Identity $user -NewAlias 'i:0#.w|xenonatom\jdoe' -IgnoreSid
```

The downside to [Move-SPUser](https://docs.microsoft.com/en-us/powershell/module/sharepoint-server/move-spuser?view=sharepoint-ps) is you must provide a valid site where the user resides in the UIL. This is why I opt for [stsadm](https://docs.microsoft.com/en-us/previous-versions/office/sharepoint-2007-products-and-technologies/cc262141(v%3doffice.12)). They go through the same code base, so there should be no concerns about using one over the other.

```powershell
stsadm -o migrateuser -oldlogin 'i:0#.w|lab\jdoe' -newlogin 'i:0#.w|xenonatom\jdoe' -ignoresidhistory
```

Notice we use the switches to ignore the user's SID history. This is required when using Windows Claims, but for those on Windows Classic (where you would not specify the i:0#.w|), you do not need to use this switch. Note that although you only specify a single Web URL with Move-SPUser, the migration is farm-wide. Also note that all migrates are farm-wide and it is not possible to provide a narrower scope.

At this point, it is simply a matter of your domain admin migrating user accounts and running one of the two above commands when users are migrated. When the migration has completed, remember to remove the SearchActiveDirectoryDomain object for the source forest (and both can be dropped if the SharePoint service accounts reside in the target forest). You can also remove the People Picker filter as well as the HideInactiveProfiles on the farm object.

One last thing of note, the client-side People Picker included in SharePoint 2013+ does cache entries in your browser! If you perform a migration and have previously looked up a user in the People Picker but still find the 'old' user, empty your browser cache, use InPrivate mode, or try another browser to verify that the People Picker is operating correctly and only finding the newly enabled target account and not the disabled source account.

Forest migrations are a lot of work and it is important for those in the decision making process to recognize the limitations with regards to SharePoint in your environment during the planning phase of a migration. If you have any questions, feel free to contact me on Twitter, post in SharePoint Stack Exchange, TechNet, or in the comments below.