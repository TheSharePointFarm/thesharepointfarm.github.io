---
layout: post
title: SharePoint Server 2016 Audiences with MIM
tags: [mim,sp2016]
---

EDIT 3/14/2017: This has been fixed as of the February 2017 Public Update. For more information, see [Microsoft Identity Manager NoILMUsed Bug Fixed]({% post_url 2017-02-21-microsoft-identity-manager-noilmused-bug-fixed %}).

SharePoint Server 2016 Audiences based on the Member Of Operator with MIM do not work out of the box. On the SharePoint side, within the Profile database, the upa.MemberGroup column named MemberCount only shows a count of 0. In addition, the table upa.UserMemberships is blank.

There are two changes need to be made, one to the MIM Active Directory Management Agent. On the ADMA, create a new flow for Groups.

Under Configure Attribute Flow, select Group, then sAMAccountName. In the metaverse, select group and accountName.

![ADMAGroups](/assets/images/2016/04/ADMAGroups.png)

The next step is to set the UPSA to use the External Identity Manager. You've already done this through the User Profile Service Application, but that doesn't trigger the necessary settings to populate Active Directory group membership within the Profile database.

```powershell
$sa = Get-SPServiceApplication | ?{$_.TypeName -eq "User Profile Service Application"}
$sa.NoILMUsed = 1
$sa.Update()
```

Once this is completed, perform a full synchronization using Start-SharePointSync. You will then be able to create an Audience that leverages the Member Of Operator. From there, you'll be able to search for groups when creating an Audience based off of MemberOf in addition to being able to Compile the Audience.