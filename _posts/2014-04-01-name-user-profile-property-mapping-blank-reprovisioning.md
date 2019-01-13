---
layout: post
title: Name User Profile Property Mapping Blank After Reprovisioning
tags: [sp2013]
---

The Name property in the User Profile Service Application (also known as PreferredName) controls how a user's friendly name will appear within the UPA (and by extension, SharePoint). By default, this property is mapped to a Synchronization Connection's displayName attribute.

![NameProperty](/assets/images/2014/04/NameProperty.png)

Changing the Name property from the _displayName_ attribute to another attribute will cause the Name property's mapping to disappear on a restart of the User Profile Synchronization Service (for example, during reprovisioning after a full farm backup has taken place). If the SharePoint administrator then attempts to change the Name property mapping back to the _displayName_ attribute, the mapping will also disappear during reprovisioning.

The reasoning for this happening is that by default, SharePoint creates the Name property with the following flow using the _displayName_ attribute:

![displayNameRulesExtension](/assets/images/2014/04/displayNameRulesExtension.png)

When changing the Name property to a new attribute, the flow is changed like so (in this particular case, the displayName attribute mapping was remove, and re-added, returning back to Manage User Properties in between each edit):

![displayNameDirectMapping](/assets/images/2014/04/displayNameDirectMapping.png)

Note the metaverse attribute is now _PreferredName_. Because this is a Direct mapping, it no longer correlates to the Name property in use by the UPA. Any changes to the Name property moving forward will change this new Direct mapping, rather than the proper Extension mapping. As a result, the Name property will appear to have no mapping:

![NamePropertyMissing](/assets/images/2014/04/NamePropertyMissing.png)

To resolve this issue, delete and recreate the Synchronization Connection. The default mapping will be restored and will not disappear on UPSS reprovisioning. As a result, I'd recommend not adjusting the mapping of this property.

This issue was validated in SharePoint 2013 SP1.