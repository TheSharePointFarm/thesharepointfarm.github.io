---
layout: post
title: SharePoint 2013 and Active Directory Lightweight Directory Services – Small Changes Required
tags: [ad-lds,news,sp2013]
---

My article covering [SharePoint 2010 with AD LDS]({% post_url 2012-01-03-sharepoint-2010-and-active-directory-lightweight-directory-services-better-together %}) fortunately only requires one small change to be SharePoint 2013-compatible!  The version of the Microsoft.Office.Server binary needs to be updated from 14.0.0.0 to 15.0.0.0 in the appropriate configuration files:

Web Application web.config:

```xml
<membership defaultProvider="i">
	<providers>
	<add name="i" type="Microsoft.SharePoint.Administration.Claims.SPClaimsAuthMembershipProvider, Microsoft.SharePoint, Version=15.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c" />
	<add name="FabrikamMembership" type="Microsoft.Office.Server.Security.LdapMembershipProvider, Microsoft.Office.Server, Version=15.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c" server="adlds01.nauplius.local" port="636" useSSL="true" userDNAttribute="distinguishedName" useDNAttribute="true" userNameAttribute="mail" userContainer="CN=SharePoint,DC=Fabrikam,DC=local" userObjectClass="user" userFilter="(ObjectClass=user)" scope="Subtree" otherRequiredUserAttributes="sn,givenname,cn" />
	</providers>
</membership>
<roleManager defaultProvider="c" enabled="true" cacheRolesInCookie="false">
	<providers>
	<add name="c" type="Microsoft.SharePoint.Administration.Claims.SPClaimsAuthRoleProvider, Microsoft.SharePoint, Version=15.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c" />
	<add name="FabrikamRole" type="Microsoft.Office.Server.Security.LdapRoleProvider, Microsoft.Office.Server, Version=15.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c" server="adlds01.nauplius.local" port="636" useSSL="true" enableSearchMethods="true" groupContainer="CN=SharePoint,DC=Fabrikam,DC=local" groupNameAttribute="cn" groupNameAlternateSearchAttribute="cn" groupMemberAttribute="member" userNameAttribute="mail" dnAttribute="distinguishedName" useUserDNAttribute="true" scope="Subtree" userFilter="&amp;(objectClass=user)(objectCategory=person)" groupFilter="&amp;(objectCategory=Group)(objectClass=group)" />
	</providers>
</roleManager>
```

Security Token Service web.config:

```xml
<system.web>
     <membership>
         <providers>
             <add name="FabrikamMembership"
                  type="Microsoft.Office.Server.Security.LdapMembershipProvider, Microsoft.Office.Server, Version=15.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c"
                  server="adlds01.nauplius.local"
                  port="636"
                  useSSL="true"
                  enableSearchMethods="true" 
                  userDNAttribute="distinguishedName"
                  userNameAttribute="mail"
                  userContainer="CN=SharePoint,DC=Fabrikam,DC=local"
                  userObjectClass="user"
                  userFilter="(ObjectClass=*)"
                  scope="Subtree"
                  otherRequiredUserAttributes="sn,givenname,cn" />
         </providers>
     </membership>
     <roleManager enabled="true">
         <providers>
             <add name="FabrikamRole"
                  type="Microsoft.Office.Server.Security.LdapRoleProvider, Microsoft.Office.Server, Version=15.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c"
                  server="adlds01.nauplius.local"
                  port="636"
                  useSSL="true"
                  enableSearchMethods="true" 
                  groupContainer="CN=SharePoint,DC=Fabrikam,DC=local"
                  groupNameAttribute="cn"
                  groupNameAlternateSearchAttribute="cn"
                  groupMemberAttribute="member"
                  userNameAttribute="mail"
                  dnAttribute="distinguishedName"
                  useUserDNAttribute="true"
                  userFilter="&amp;(objectClass=user)(objectCategory=person)"
                  groupFilter="&amp;(objectCategory=Group)(objectClass=group)"
                  scope="Subtree" />
         </providers>
     </roleManager>
</system.web>
```

Central Administration web.config:

```xml
<roleManager>
	<providers>
		<add name="FabrikamRole" type="Microsoft.Office.Server.Security.LdapRoleProvider, Microsoft.Office.Server, Version=15.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c" server="adlds01.nauplius.local" port="636" useSSL="true" enableSearchMethods="true" groupContainer="CN=SharePoint,DC=Fabrikam,DC=local" groupNameAttribute="cn" groupNameAlternateSearchAttribute="cn" groupMemberAttribute="member" userNameAttribute="mail" dnAttribute="distinguishedName" useUserDNAttribute="true" scope="Subtree" userFilter="&amp;(objectClass=user)(objectCategory=person)" groupFilter="&amp;(objectCategory=Group)(objectClass=group)" />
	</providers>
</roleManager>
<membership>
	<providers>
		<add name="FabrikamMembership" type="Microsoft.Office.Server.Security.LdapMembershipProvider, Microsoft.Office.Server, Version=15.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c" server="adlds01.nauplius.local" port="636" useSSL="true" userDNAttribute="distinguishedName" useDNAttribute="true" userNameAttribute="mail" userContainer="CN=SharePoint,DC=Fabrikam,DC=local" userObjectClass="user" userFilter="(ObjectClass=user)" scope="Subtree" otherRequiredUserAttributes="sn,givenname,cn" />
	</providers>
</membership>
```