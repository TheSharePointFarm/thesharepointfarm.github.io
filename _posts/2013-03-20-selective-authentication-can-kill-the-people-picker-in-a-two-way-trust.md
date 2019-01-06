---
layout: post
title: Selective Authentication can Kill the People Picker in a Two-Way Trust
tags: [sp2010]
---

One thing to watch out for in a two-way trust scenario with the People Picker.  Typically with the People Picker in a two-way forest trust, you do not have to make any changes to SharePoint to resolve users in the target trusted forest.

However, if the Administrators of the target forest have chosen to implement Selective Authentication, you'll run into an issue where by users cannot be resolved in the remote domain.

![SelectiveAuthentication](/assets/images/2013/03/SelectiveAuthentication.png)

The ULS log will display the following (this was a search for 'user'):

```csharp
Error in searching user 'user' : System.DirectoryServices.DirectoryServicesCOMException (0x8007203B): A local error has occurred. at System.DirectoryServices.DirectoryEntry.Bind(Boolean throwIfFail) at System.DirectoryServices.DirectoryEntry.Bind() at System.DirectoryServices.DirectoryEntry.get_AdsObject() at System.DirectoryServices.DirectorySearcher.FindAll(Boolean findMoreThanOne) at Microsoft.SharePoint.WebControls.PeopleEditor.SearchFromGC(SPActiveDirectoryDomain domain, String strFilter, String[] rgstrProp, Int32 nTimeout, Int32 nSizeLimit, SPUserCollection spUsers, ArrayList& rgResults) at Microsoft.SharePoint.Utilities.SPUserUtility.SearchAgainstAD(String input, SPActiveDirectoryDomain domainController, SPPrincipalType scopes, SPUserCollection usersContainer, Int32 maxCount, String customQuery, String customFilter, TimeSpan searchTimeout, Boolean& reachMaxCount) at Microsoft.SharePoint.Utilities.SPActiveDirectoryPrincipalResolver.SearchPrincipals(String input, SPPrincipalType scopes, SPPrincipalSource sources, SPUserCollection usersContainer, Int32 maxCount, Boolean& reachMaxCount) at Microsoft.SharePoint.Utilities.SPUtility.SearchPrincipalFromResolvers(List`1 resolvers, String input, SPPrincipalType scopes, SPPrincipalSource sources, SPUserCollection usersContainer, Int32 maxCount, Boolean& reachMaxCount, Dictionary`2 usersDict).
```

Additionally, a Network Monitor trace from the SharePoint server will show the following entry at the time of the attempted search:

![SelectiveAuthKerberos](/assets/images/2013/03/SelectiveAuthKerberos.png)

In this case, 10.10.20.18 is the SharePoint server, a member of Nauplius.local, and 10.10.20.25 is the Domain Controller for Contoso.local.  With Selective authentication in place and with the Application Pool service account not having any explicit rights in Contoso.local to resolve users, we receive KDC_ERR_POLICY.  In order to resolve this issue while maintaining Selective Authentication, add the Application Pool account (or any account needing to resolve users in the remote domain using the People Picker) the Allowed to Authenticate right in the Domain Controller computer objects in the remote domain.  Make sure to do this for each Domain Controller computer object in the remote domain that SharePoint Server is able to authenticate to.

![AllowedtoAuth](/assets/images/2013/03/AllowedtoAuth.png)

Given no Kerberos tickets were previously granted by Contoso.local to, in the above example, to NAUPLIUS\s-sp2010apppool, there is no need to log the account out of the SharePoint server(s) (in other words, iisreset).