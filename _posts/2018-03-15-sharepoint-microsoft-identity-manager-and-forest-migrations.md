---
layout: post
title: SharePoint, Microsoft Identity Manager, and Forest Migrations
tags: [mim,sp2016]
---

This article will walk you through configuring SharePoint and Microsoft Identity Manager to support Active Directory forest migrations.

'I'm in process of a forest migration and ran into a few complications with Microsoft Identity Manager and SharePoint Server 2016. Although I had Management Agents to both Forests to import User objects, when performing the user migration the SharePoint Profile was deleted and recreated instead of being joined. This is unlike SharePoint Server 2010 and 2013 where the SharePoint Admin didn't have to make any special configuration with regards to the User Profile when leveraging the User Profile Synchronization Service.

If you're not familiar with a forest migration, the process usually breaks down like this:

* Create a two-way trust
* Disable SID Filtering/Enable SID History
* Synchronize all objects you intend to migrate placing objects in a disabled state in the destination domain
* Migrate a user, disabling the source domain account and enabling the target domain account

There are variants to how this is done, but these are the big and important pieces. The two way trust allows users to access resources cross-forest without issues. The sIDHistory stamps the migrated user object in the destination forest with the objectSid value of the source object; this would enable preservation of ACLs on NTFS-based resources, as an example (it doesn't facilitate SharePoint secured resources or Profiles). The last piece is to disable the source account and enabling the target account facilitate security -- it breaks the Microsoft security model to have two objects with the same SID (or in our case, objectSid in source, sIDHistory in destination).

To start our migration journey, let's begin with SharePoint. The first step is to modify the People Picker to enable friendly return results cross-forest and filter out disabled accounts. This is easily done through PowerShell and must be done for each Web Application in use where you need to resolve objects cross-forest.

First, filter disabled user objects.

```powershell
$wa = Get-SPWebApplication https://sharepoint.example.com
$wa.PeoplePickerSettings.ActiveDirectoryCustomFilter = '(!userAccountControl:1.2.840.113556.1.4.803:=2)'
$wa.Update()
```

Next, we need to point the People Picker at both domains. This did not used to be required with SharePoint 2010 but the updated People Picker in 2013 and above does require it in order to display the displayName of the user rather than just the username itself.

```powershell
$wa = Get-SPWebApplication https://sharepoint.cobaltatom.com
#Searcher object for the source domain
$seacher = New-Object Microsoft.SharePoint.Administration.SPPeoplePickerSearchActiveDirectoryDomain
$seacher.DomainName = "lab.cobaltatom.com"
$seacher.ShortDomainName = "LAB" #Optional
$seacher.IsForest = $true

#Searcher object for the target domain
$seacher2 = New-Object Microsoft.SharePoint.Administration.SPPeoplePickerSearchActiveDirectoryDomain
$seacher2.DomainName = "xenonatom.com"
$seacher2.ShortDomainName = "XENONATOM" #Optional
$seacher2.IsForest = $true

$wa.PeoplePickerSettings.SearchActiveDirectoryDomains.Add($seacher)
$wa.PeoplePickerSettings.SearchActiveDirectoryDomains.Add($seacher2)
$wa.Update()
```

At this point, within SharePoint, you should be able to find users from both forests via the People Picker.

Finally, we need to hide disabled accounts from the farm. Add the following farm property.

```powershell
$farm = Get-SPFarm
$farm.Properties.Add("HideInactiveProfiles","true")
$farm.Update()
```

There is no requirement to restart any service nor perform an IISReset. These settings are effective immediately.

The next portion of this setup we move to Microsoft Identity Manager. This post assumes you have already set up SharePoint with MIM and are synchronizing both forests into the MIM metaverse with two Active Directory Management Agents. We'll need to make some changes to MIM, the metaverse, and the Management Agents. The first step in this process is to define the attribute we are going to join the person objects on (in the metaverse, Users from Active Directory map to the person object). We need to use an attribute that is indexable and single value. This rules out mapping the ObjectSID attribute to the sidHistory attribute as sidHistory is mutli-valued and non-indexable. You can consider the sAMAccountName or mail attributes, but if your company changes sAMAccountName values when someone changes their name, that will also not work. The mail attribute may also change for the same reason, or perhaps it doesn't have any value at all! This leaves few values left that are unique. Fortunately every object has an ObjectGUID attribute which is unique but we cannot leverage it directly as the attribute's value will not be the same as a user is migrated to the target forest (it is unique, after all...).

The route I took was to instead copy the ObjectGUID value from a user into one of the Exchange-provisioned extension attributes; more specifically, extensionAttribute1. This can be easily accomplished via PowerShell:

```powershell
$users = Get-ADUser -Filter 'enabled -eq $true'

foreach($user in $users) {
    $guid = $user | select ObjectGuid
    Set-ADUser $user -Replace @{extensionAttribute1 = $guid.ObjectGuid.ToString()}
}
```

In the Synchronization Service Manager, go to the Metaverse Designer. The first thing we need to do is configure the Object Deletion rule. Highlight person in the Object types pane and click Configure Object Deletion Rule in the Actions pane. Change the deletion rule to use the _target_ forest Management Agent.

![objectdeletionrule](/assets/images/2018/03/objectdeletionrule.png)

With person still highlighted, Add a new Attribute in the Actions pane on the right. In the Add Attribute To Object Type window, click New attribute. This is the attribute we're going to synchronize; in my case, extensionAttribute1. Create the attribute with a type of String (indexable) and mark it as Indexed. Click OK twice to return to the Metaverse Designer.

![newattribute](/assets/images/2018/03/newattribute.png)

Before coming back to the Metaverse Designer, switch over to the Management Agents. We will repeat this process on both Active Directory Management Agents. We will not touch the SharePoint Management Agent. In each AD MA, we will be adding the extensionAttribute1 as a flow and creating a join rule for our User object. Under Select Attributes, click Show All and find the attribute you've selected to use; in my case, extensionAttribute1.

![addexttoma](/assets/images/2018/03/addexttoma.png)

Under Configure Join and Projection Rules, click on the user data source object type and click New Join Rule. Add the rule of:

```
Data source attribute: extensionAttribute1

Mapping type: Direct

Metaverse object type: person

Metaverse attribute: extensionAttribute1
```

Click Add Condition when the attributes are highlighted on either side.

![joinandproject-1](/assets/images/2018/03/joinandproject-1.png)

The end result looks like this.

![joinandproject-2]J(/assets/images/2018/03/joinandproject-2.png)

The last item to change in the Management Agent is to add the attribute flow. Under Configure Attribute Flow, add a new flow for the user to person object type as shown in the screenshot. Don't forget to hit the New button after selecting the attribute flow, otherwise it will not be recreated.

![attributeflow](/assets/iamges/2018/03/attributeflow.png)

Repeat these steps for the other AD MA. Remember the flow direction is identical for both MAs; that is we are going from the Data Source Attribute -> Metaverse Attribute.

The next step is to configure the attribute flow precedence in the Metaverse Designer. We want the target forest to be authoritative to attributes. Any attributes set on the target User object will "win" if there is a conflict. Sort the Import Flow column. For each attribute with a value of 2 ("2" being the number of [Active Directory] Management Agents which are leveraging that specific attribute), click Configure Attribute Flow Precedence in the Actions pane. Sort the flow order so the target forest Management Agent comes before the source forest Management Agent. Repeat this process for any attributes where the Import Flow value is 2.

![flowprecedence](/assets/images/2018/03/flowprecedence.png)

We need to disable the Provisioning rules extension. In the Synchronization Service Manager, go to Tools -> Options and deselect Enable Provisioning Rules Extension.

Prior to performing a user migration, run a full synchronization of the Management Agents. Validate via the Metaverse Search that users have the correct attribute, extensionAttribute1 in this case, populated correctly. If you have not used Metaverse Search, here's how. On the Metaverse Search tab, select person from Object Source by Type, then select the attribute you want to search by (e.g. sAMAccountName). Set the operator appropriately, then type in a value to search on; click Search. Objects returned here exist in the metaverse. You can double click the returned object to see the information in the metaverse about the object as well as which Management Agent has populated that value (if any). For example, here I searched on the sAMAccountName equals 'aaron.painter'. We can see here that the ADMA (my source forest) is the responsible Management Agent for populating all of the values, including extensionAttribute1 which will be used to join the appropriate user object in the metaverse.

![metaversesearch](/assets/images/2018/03/metaversesearch.png)

Perform a copy of all objects from the source domain to the target domain using your domain migration tooling of choice. Remember, the target object must be in a disabled state. At this point, perform a full synchronization in MIM. Prior to migrating any users, re-enable the Provisioning rules extension in the Tools -> Options menu.

From here, you can start to test user migrations and validating that they're joining properly in the metaverse as well as preserving the User Profile in the SharePoint UPSA. Don't forget to disable the source account when you migrate! You will also still need to validate SharePoint permissions as this process will not fix that (though interestingly enough, the UPSA does kick off a user migration, but I've seen issues where it doesn't work as well as an `stsadm -o migrateuser`/`Move-SPUser`).

Now you should be ready to perform that forest migration with SharePoint leveraging MIM. Enjoy the project :-)