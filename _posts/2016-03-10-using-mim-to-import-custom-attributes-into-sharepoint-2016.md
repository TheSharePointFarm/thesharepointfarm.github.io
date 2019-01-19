---
layout: post
title: Using MIM to Import Custom Attributes into SharePoint 2016
tags: [sp2016]
---

# Microsoft Identity Manager Series

Part 1: [Automating MIM User Profile Synchronization with SharePoint 2016]({% post_url 2016-03-09-automating-mim-user-profile-synchronization-with-sharepoint-2016 %})

Part 2: _Using MIM to Import Custom Attributes into SharePoint 2016_

Part 3: [Using MIM to Export Custom Attributes from SharePoint 2016]({% post_url 2016-03-11-using-mim-to-export-attributes-from-sharepoint-2016 %})

Part 4: [Default MIM to SharePoint 2016 Attribute Mappings]({% post_url 2016-03-13-default-mim-to-sharepoint-2016-attribute-mappings %})

Part 5: [Basic MIM Configuration to Support SharePoint 2016]({% post_url 2016-03-15-basic-mim-configuration-support-sharepoint-2016 %})

Part 6: [Scoping the Active Directory Management Agent in MIM]({% post_url 2016-03-16-scoping-active-directory-management-agent-mim %})

Using MIM to import custom attributes into SharePoint 2016 requires a variety of changes to MIM that may be hard to understand until you get a hold of the terminology. This post assumes you have previously set up MIM to import profiles into SharePoint 2016 already. Here, we will create a new Profile Property in the User Profile Service Application, configure the Active Directory Management Agent (ADMA) to import the property into the Metaverse, and finally configure the SharePoint Management Agent (SPMA) to export the property to the SharePoint User Profile Service.

First, define your attribute in Active Directory that you want to import. This attribute can be a string, multi-value string, reference, etc. Note the size limit of the attribute, for example, 'drink' is limited to 256 characters while 'company' is a maximum of 64 characters. From the User Profile Service Application in Central Administration, define your new Profile Property to match the characteristics of the Active Directory attribute. For this example, I will use the attribute 'employeeType', which is a single-value string with a maximum length of 256 characters. The SharePoint User Profile Property is named 'EmployeeType' and set to Optional, as not all users will have this attribute populated.

Using the Synchronization Service manager, under Management Agents, we'll edit the ADMA. Under Select Attributes, click on "Show All" so we can see the 'employeeType' attribute; check the attribute.

Moving onto Attribute Flow, we are going to use the following configuration:

```
Data source object type: user
Data source attribute: employeeType
Metaverse object type: person
Metaverse attribute: employeeType
Mapping Type: Direct
Flow Direction: Import
```

![ADMA-ET-Import](/assets/images/2016/03/ADMA-ET-Import.png)

Once those options have been selected, click New. This will create the mapping from Active Directory into the Metaverse. Click OK, and the attribute is now configured for the ADMA. For the SPMA, if we go to edit the MA immediately, we will not see our new 'EmployeeType' Profile Property under Select Attributes. This is because we need to refresh the schema. The same would apply if the attribute was new in Active Directory, as well. Right click on the SPMA and select Refresh Schema. You will need to supply the password for the account configured to connect back to SharePoint 2016. Once the schema has been committed, we can now edit the SPMA. Go to Select Attributes and you will now notice 'EmployeeType' is present. Select it. Now, let's configure the Attribute Flow with this configuration:

```
Data source object type: user
Data source attribute: EmployeeType
Metaverse object type: person
Metaverse attribute: employeeType
Mapping Type: Direct
Flow Direction: Export, Allow Nulls
```

![SPMA-ET-Export](/assets/images/2016/03/SPMA-ET-Export.png)

Note how we label this as an export attribute. That is because we are exporting from the Metaverse to SharePoint. Click New once the Attribute Flow has been configured properly.

The last step is to validate that it works. You must run a full synchronization when creating a new Attribute Flow, just like you would in SharePoint 2010 or 2013 when using the User Profile Synchronization Service with a new Profile Property. To do this, we will simply use `Start-SharePointSync` from the [Office PnP UserProfile.MIMSync](https://github.com/OfficeDev/PnP-Tools/tree/master/Solutions/UserProfile.MIMSync) solution.

And now you know how to import new attributes into the User Profile Service Application using Microsoft Identity Manager.