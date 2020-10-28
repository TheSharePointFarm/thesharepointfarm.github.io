---
layout: post
title: Creating a Microsoft Team using PowerShell on an Existing Group
tags: [teams]
---

If you have a Microsoft 365 Group and you want to add a Microsoft Team to the Group, you can use the Microsoft Graph PowerShell to do so.

To get started, we need to install the [Microsoft Graph PowerShell](https://www.powershellgallery.com/packages/Microsoft.Graph/) from the PowerShell Gallery. Note that this is a rather large set of modules and may take some time to install after downloading.

This particular method of connecting and creating a Team requires Global Administrator or the Teams Admin role as we are using Delegated permissions rather than Application permissions. You will need the ID of the Microsoft 365 Group. There are a variety of ways to get this, including from Graph PowerShell, SharePoint Online PowerShell, and so forth. You can also get it from [Azure Active Directory UI](https://portal.azure.com/#blade/Microsoft_AAD_IAM/GroupsManagementMenuBlade/AllGroups).

```powershell
Install-Module Microsoft.Graph -Scope CurrentUser #Remove CurrentUser if you want to install machine-wide
Connect-MgGraph -Scopes "Group.ReadWrite.All, Directory.ReadWrite.All"
#Complete Device login
$body =
'{
  "memberSettings": {
    "allowCreateUpdateChannels": true,
    "allowDeleteChannels": true,
    "allowAddRemoveApps": true,
    "allowCreateUpdateRemoveTabs": true,
    "allowCreateUpdateRemoveConnectors": true
  },
  "messagingSettings": {
    "allowUserEditMessages": true,
    "allowUserDeleteMessages": true,
    "allowOwnerDeleteMessages": true,
    "allowTeamMentions": true,
    "allowChannelMentions": true
  },
  "funSettings": {
    "allowGiphy": true,
    "giphyContentRating": "strict",
    "allowStickersAndMemes": true,
    "allowCustomMemes": true
  }
}' #define the options for the Team. See below for available options
Invoke-MgGraphRequest -Method PUT -Uri https://graph.microsoft.com/v1.0/groups/<M365GroupId>/team -ContentType "application/json" -Body $body #Create the Team
```

Once invoked, you should see a response similar to this:

```
Name                           Value
----                           -----
displayName                    Change Request
classification
guestSettings                  {allowCreateUpdateChannels, allowDeleteChannels}
messagingSettings              {allowUserEditMessages, allowOwnerDeleteMessages, allowChannelMentions, allowUserDeleteMessages...}
@odata.context                 https://graph.microsoft.com/v1.0/$metadata#teams/$entity
specialization
webUrl                         https://teams.microsoft.com/l/team/19:48a42ce16238439dbe1fb02ba8e8288d%40thread.tacv2/conversations?groupId=057697b5-57f8-4f1d-8b87-7347faa91cf6&tenantId=95920ba6-0edd-4d0c-b...
memberSettings                 {allowDeleteChannels, allowCreatePrivateChannels, allowCreateUpdateChannels, allowCreateUpdateRemoveTabs...}
visibility
id                             057697b5-57f8-4f1d-8b87-7347faa91cf6
description                    Change Request
funSettings                    {allowCustomMemes, allowGiphy, allowStickersAndMemes, giphyContentRating}
isArchived
isMembershipLimitedToOwners    False
discoverySettings
internalId                     19:48a42ce16238439dbe1fb02ba8e8288d@thread.tacv2
```

And that's it! To see the available Team options you can set in the `$body` variable, you can execute a GET request on the Team.

```powershell
$team = Invoke-MgGraphRequest -Method GET -Uri https://graph.microsoft.com/v1.0/teams/057697b5-57f8-4f1d-8b87-7347faa91cf6
$team.guestSettings
$team.messagingSettings
#and so on...
```

For additional information, see the Microsoft Graph documentation on [Create a team from group](https://docs.microsoft.com/graph/api/team-put-teams) API.
