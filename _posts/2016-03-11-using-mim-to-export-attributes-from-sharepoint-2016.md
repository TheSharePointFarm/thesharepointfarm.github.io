---
layout: post
title: Using MIM to Export Attributes from SharePoint 2016
tags: [sp2016]
---

**Microsoft Identity Manager Series**

Part 1: [Automating MIM User Profile Synchronization with SharePoint 2016]({% post_url 2016-03-09-automating-mim-user-profile-synchronization-with-sharepoint-2016 %})

Part 2: [Using MIM to Import Custom Attributes into SharePoint 2016]({% post_url 2016-03-10-using-mim-to-import-custom-attributes-into-sharepoint-2016 %})

Part 3: _Using MIM to Export Custom Attributes from SharePoint 2016_

Part 4: [Default MIM to SharePoint 2016 Attribute Mappings]({% post_url 2016-03-13-default-mim-to-sharepoint-2016-attribute-mappings %})

Part 5: [Basic MIM Configuration to Support SharePoint 2016]({% post_url 2016-03-15-basic-mim-configuration-support-sharepoint-2016 %})

Part 6: [Scoping the Active Directory Management Agent in MIM]({% post_url 2016-03-16-scoping-active-directory-management-agent-mim %})

Using MIM to Export Attributes from SharePoint 2016 involves a few more steps than using MIM to import attributes to SharePoint 2016. We will not only need to manipulate the Active Directory Management Agent (ADMA) and SharePoint Management Agent (SPMA), but also add the attribute to the Metaverse as well as create an Export Run Profile for the ADMA.

First step is to define your attribute in the User Profile Service Application as well as what attribute it will be exported to in Active Directory. For this example, we will use the Profile Property 'FavoriteDrink' which will export to the Active Directory attribute 'drink'. The Active Directory attribute 'drink' has a character limit of 256 characters (yes, this is an out of the box attribute!). So, I will make a new Profile Property attribute named 'FavoriteDrink', string (single value), 64 characters maximum, optional, and editable by the user. One other thing to note about this attribute is it is not part of the user class in the Active Directory schema, so it must be added to the user class via Active Directory Schema Manager.

As before, the account used to synchronize with Active Directory must be able to write the attribute in question. Make sure this is configured via delegation prior to continuing. Remember, you can delegate write permissions to specific attributes! You do not need to give your synchronization account Domain Administrator rights :-) (but you may have to use ADSI Edit to grant those rights).

Because there is no Metaverse attribute to hold the value of 'drink', we'll need to create one. In the Synchronization Service manager, under Metaverse Designer, highlight 'person' in Object Types. On the right, click Add Attribute, and then because it isn't part of the available existing attributes, click on New Attribute. Give it a name, 'drink', and provide it a type. The Active Directory attribute 'drink' is a non-index string, so I'll select 'String (non-indexable)'. It also does not support multiple values, so there is no need to check that box. Click OK to create the attribute in the Metaverse, and OK again. It will now appear in the lower pane with the rest of the attributes.

![MetaverseAttribute](/assets/images/2016/03/MetaverseAttribute.png)

The next step is to configure the SPMA. Under Management Agents, highlight the SPMA and click Refresh Schema. This is required as we just added a brand new Profile Property. Once the schema has refreshed successfully, go into the Properties of the SPMA. Click Select Attribute, and find 'FavoriteDrink'. Next, under Configure Attribute Flow, create the following flow to _import_ 'FavoriteDrink' into the Metaverse attribute 'drink'.

```
Data source object type: user
Data source attribute: FavoriteDrink
Metaverse object type: person
Metaverse attribute: drink
Mapping Type: Direct
Flow Direction: Import
```

![SPMA-D-Import](/assets/images/2016/03/SPMA-D-Import.png)

Click New once the proper settings are selected. This completes the SPMA configuration. Next up, ADMA. As I had to add the 'drink' attribute to the user class in Active Directory Schema Manager, I also must refresh the schema on the Management Agent. If you used an existing property, for example, 'company' or 'extentionAttribute1', this would not be required. Under Select Attributes, I need to 'Show All' in order to see the 'drink' attribute. Next up, under Configure Attribute Flow, I will configure an export attribute flow from the Metaverse into Active Directory, allowing for null values as not all users will have populated the Profile Property in SharePoint.

```
Data source object type: user
Data source attribute: drink
Metaverse object type: person
Metaverse attribute: drink
Mapping Type: Direct
Flow Direction: Export, Allow Nulls
```

![ADMA-D-Export](/assets/images/2016/03/ADMA-D-Export.png)

Click New once it is properly configured. Exit the ADMA Properties, there is one last step. We must create a Run Profile for the ADMA. Out of the box, and hopefully this is corrected in the very near future, the ADMA does not have an Export Run Profile. This means we would not be able to export any properties from the Metaverse into Active Directory, but setting one up is easy. With the ADMA selected, click on Configure Run Profiles. Click New Profile, name it Export, and select Export under the step type. Finish out the wizard leaving the remainder settings at their defaults.

Have a user populate the Favorite Drink Profile Property in their User Profile. Run the full synchronization (remember, we created a new property and reconfigured the MAs!) using Start-SharePointSync. Once that has completed, the change will be staged in the Metaverse, waiting to run the new Export Run Profile. To execute that specific run profile, use Start-ManagementAgent ADMA -RunProfile EXPORT. This will export the new 'drink' property from the Metaverse to Active Directory, and we can then see the value for the test user using ADSI Edit.

And now you know how to export profile properties from SharePoint 2016 via Microsoft Identity Manager into Active Directory!