---
layout: post
title: Default MIM to SharePoint 2016 Attribute Mappings
tags: [sp2016,mim]
---

# Microsoft Identity Manager Series

Part 1: [Automating MIM User Profile Synchronization with SharePoint 2016]({% post_url 2016-03-09-automating-mim-user-profile-synchronization-with-sharepoint-2016 %})

Part 2: [Using MIM to Import Custom Attributes into SharePoint 2016]({% post_url 2016-03-10-using-mim-to-import-custom-attributes-into-sharepoint-2016 %})

Part 3: [Using MIM to Export Custom Attributes from SharePoint 2016]({% post_url 2016-03-11-using-mim-to-export-attributes-from-sharepoint-2016 %})

Part 4: _Default MIM to SharePoint 2016 Attribute Mappings_

Part 5: [Basic MIM Configuration to Support SharePoint 2016]({% post_url 2016-03-15-basic-mim-configuration-support-sharepoint-2016 %})

Part 6: [Scoping the Active Directory Management Agent in MIM]({% post_url 2016-03-16-scoping-active-directory-management-agent-mim %})

This is the default attribute mapping for the Active Directory Management Agent (ADMA), MIM Metaverse, and SharePoint Management Agent (SPMA). Note that not all attributes have flows as they're used for rule extensions, which are rules defined in code.

ADMA | Active Directory Object Type | Metaverse | Metaverse Object Type | SharePoint Object Type | SPMA
--- | --- | --- | --- | --- | ---
sAMAccountName|user|accountName,sAMAccountName|person|user|ProfileIdentifier,UserName
sAMAccountName|user|accountName,sAMAccountName|contact|contact|ProfileIdentifier
name|domainDNS|dc|domain|N/A|N/A
department|user|department|person|user|Department,SPS-Department
description|group|description|group|group|Description
displayName|group,user|displayName|contact,group,person|contact,group,user|PreferredName
dc|N/A|N/A|N/A|N/A|N/A
<dn>|domainDNS|distinguishedName|domain|N/A|N/A
<dn>|user|distinguishedName|contact,person|contact,user|PreferredName,ProfileIdentifier,SPS-DistinguishedName
<dn>|group|distinguishedName|group|group|PreferredName,ProfileIdentifier
givenName|user|firstName|person|user|FirstName
groupType|group|groupType|group|group|GroupName
sn|user|lastName|person|user|LastName
mail|user|mail|person|user|WorkEmail
mail|user|mail|group|group|MailNickName
manager|user|manager|person|user|Manager
member|group|member|group|group|Member
msDS-SourceObjectDN|user|msDS-SourceObjectDN|contact,person|contact,user|SPS-SourceObjectDN
nCName|crossRef|nCName|crossref|N/A|N/A
nETBIOSName|crossRef|nETBiosName|crossref|N/A|N/A
objectSid|user|objectSid|group,person|group,user|SID
thumbnailPhoto|user|photo|person|user|Picture
physicalDeliveryOfficeName|user|physicalDeliveryOfficeName|person|user|Office
N/A|user|SPS-ClaimProviderID|person|user|SPS-ClaimProviderID
N/A|user|SPS-ClaimProviderType|person|user|SPS-ClaimProviderType
N/A|crossRef,user|SPS-ObjectExists|contact,crossref,person|N/A|N/A
msDS-PhoneticDisplayName|user|SPS-PhoneticDisplayName|person|user|SPS-PhoneticDisplayName
msDS-PhoneticFirstName|user|SPS-PhoneticFirstName|person|user|SPS-PhoneticFirstName
msDS-PhoneticLastName|user|SPS-PhoneticLastName|person|user|SPS-PhoneticLastName
proxyAddresses|user|SPS-SipAddress|person|user|SPS-SipAddress
userPrincipalName|user|SPS-UserPrincipalName|person|user|SPS-UserPrincipalName
telephoneNumber|user|telephoneNumber|person|user|WorkPhone
title|user|title|person|user|SPS-JobTitle,Title
N/A|domainDNS|type|domain|N/A|N/A
url|user|url|group,person|group|Url
wWWHomePage|user|wWWHomePage|person|user|PublicSiteRedirect