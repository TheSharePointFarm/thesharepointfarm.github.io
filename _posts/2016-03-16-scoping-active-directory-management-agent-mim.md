---
layout: post
title: 
tags: [sp2016]
---

# Microsoft Identity Manager Series

Part 1: [Automating MIM User Profile Synchronization with SharePoint 2016]({% post_url 2016-03-09-automating-mim-user-profile-synchronization-with-sharepoint-2016 %})

Part 2: [Using MIM to Import Custom Attributes into SharePoint 2016]({% post_url 2016-03-10-using-mim-to-import-custom-attributes-into-sharepoint-2016 %})

Part 3: [Using MIM to Export Custom Attributes from SharePoint 2016]({% post_url 2016-03-11-using-mim-to-export-attributes-from-sharepoint-2016 %})

Part 4: [Default MIM to SharePoint 2016 Attribute Mappings]({% post_url 2016-03-13-default-mim-to-sharepoint-2016-attribute-mappings %})

Part 5: [Basic MIM Configuration to Support SharePoint 2016]({% post_url 2016-03-15-basic-mim-configuration-support-sharepoint-2016 %})

Part 6: _Scoping the Active Directory Management Agent in MIM_

Many companies may not want to synchronize their entire directory to the SharePoint User Profile Service. This will guide through scoping the Active Directory Management Agent in MIM. Scoping will allow you to specify specific Organization Units to synchronize.

Using the Synchronization Service Manager, go to the Management Agents tab and double click on the Active Directory Management Agent (ADMA). In Configure Directory Partitions, make sure to highlight the Domain Naming Context partition (that is, the one that does not begin with "CN=Configuration").

![MIM_DirectoryPartitions](/assets/images/2016/03/MIM_DirectoryPartitions.png)

Click on Containers. You will be prompted for the synchronization account password to continue. Here, we can make our selections for what OUs are synchronized.

![MIM_Containers](/assets/images/2016/03/MIM_Containers.png)

On the next full synchronization, the accounts outside of the scope of the selected OUs will be deleted from the ADMA, Metaverse, and SharePoint Management Agent (SPMA), which includes the User Profile Service.

To filter out additional users, for example, those users that are disabled in Active Directory, again go back into the ADMA. In Select Attributes, select the attribute you wish to filter on. In this example, we will be filtering out disabled users, so we'll choose the attribute 'userAccountControl'. This attribute is not selected by default. Click OK to exit the ADMA. This is needed for the next step.

![MIM_FilterAttribute](/assets/images/2016/03/MIM_FilterAttribute.png)

Reopen the ADMA and go to Configure Connection Filter. In the right hand pane, highlight the 'user' object type. Select the desired attribute, or userAccountControl in this case, with "Bit on equals" with a value of 0x2 ('2'). Click Add Condition, then OK, and exit the ADMA. Start a full synchronization, and the user will be deleted from the Metaverse as well as the SPMA.

![MIM_DisableUserFilter](/assets/images/2016/03/MIM_DisableUserfilter.png)

And that is it. I hope you enjoyed this series on MIM! Any further questions about MIM as it relates to SharePoint 2016, let me know!