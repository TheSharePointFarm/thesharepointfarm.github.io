---
layout: post
title: Root Organization - It’s Your SharePoint’s Active Directory
tags: [sp2010]
---

When SharePoint 2010 hosts a Claims-based Web Application that uses Windows Authentication (NTLM or Kerberos) as well as is attached to a User Profile Service Application (that actually works…), you’ll notice with the People Entity Picker, all of your users based in Active Directory are listed under the label “Organization”.  Why?  Well, every SharePoint installation with a (working) UPA has an Organization named “Root Organization”.  This Organization contains all of your users covered by the UPA.

So, when you search for a value in the Entity Picker, the first thing that happens is that the Entity Picker enumerates all data sources (FBA, Active Directory, and so forth).  Each data source, if it can be searched, will be, attempting to identity matching values.  The values are paired with the data source.  At this point, my search for “trevor” will have a Claim from the `SPActiveDirectoryClaimProvider` filled (aka “Active Directory” in the Entity Picker).  Once the search is completed and the total count per Provider has been tallied, it is off to deconstruct the Active Directory Claims Provider results and favor the Organization in the function `Microsoft.SharePoint.Administration.Claims.SPClaimsProviderOperations.ListRedistributeHierarchy(List, Uri, SPClaimProviderManager) : List` function.  In this function, the following is called:

```csharp
SPProviderHierarchyTree result = PeoplePickerDialog.CreateClaimHierarchyProviderTree(manager.HierarchyProvider);
try
{
    List<SPProviderHierarchyTree> inHierarchies = new List<SPProviderHierarchyTree>(list);
    manager.HierarchyProvider.FillHierarchy(context, inHierarchies, result);
    manager.HierarchyProvider.FillMetadata(context, result);
    list = inHierarchies;
    list.Add(result);
```

`Manager.HierarcyProvider.FillHierarchy` walks through each entity, resolves the name in the attached UPA, and then proceeds to remove the entity from the Active Directory Claims Provider, given the entity matches one of two Claim Types, “userlogonname” or “useridentifier”.

```csharp
public override void FillMetadata(Uri context, SPProviderHierarchyTree hierarchy)
{
    if (hierarchy == null)
    {
        throw new ArgumentNullException("hierarchy");
    }
    SPServiceContext serviceContextFromUri = ClaimProvider.GetServiceContextFromUri(context);
    if ((serviceContextFromUri != null) && IsUserProfileApplicationProxyAvailableForContext(serviceContextFromUri))
    {
        List<PickerEntity> entities = new List<PickerEntity>();
        List<string> users = new List<string>();
        FillProfileEntitiesFromHierarchyElement(hierarchy, entities, users);
        if ((entities != null) && (entities.Count != 0))
        {
            UserProfileManager userManager = new UserProfileManager(serviceContextFromUri);
            OrganizationProfileManager orgManager = new OrganizationProfileManager(serviceContextFromUri);
            Hashtable userProfilesFromAccountNames = GetUserProfilesFromAccountNames(userManager, users);
            foreach (PickerEntity entity in entities)
            {
                ProfileBase profile = null;
                string normalizedHierarchyIdentifier = GetNormalizedHierarchyIdentifier(entity.HierarchyIdentifier);
                if (!string.IsNullOrEmpty(normalizedHierarchyIdentifier))
                {
                    if (IsKnownUserClaimType(entity.Claim.ClaimType))
                    {
                        profile = (ProfileBase) userProfilesFromAccountNames[normalizedHierarchyIdentifier];
                    }
                    if (profile == null)
                    {
                        profile = GetProfileFromPickerEntity(userManager, orgManager, entity);
                    }
                }
                if (profile != null)
                {
                    entity.EntityData[PeopleEditorEntityDataKeys.JobTitle] = ClaimProvider.GetProfileValue(profile, "Title");
                    entity.EntityData[PeopleEditorEntityDataKeys.Email] = ClaimProvider.GetProfileValue(profile, "WorkEmail");
                    entity.EntityData[PeopleEditorEntityDataKeys.SIPAddress] = ClaimProvider.GetProfileValue(profile, "SPS-SipAddress");
                    entity.EntityData[EntityDataKeys.Location] = ClaimProvider.GetProfileValue(profile, "SPS-Location");
                    entity.EntityData[EntityDataKeys.Picture] = ClaimProvider.GetProfileValue(profile, "PictureURL");
                }
			}
		}
	}
}
```

In `Manager.HierarcyProvider.FillMetdata`, again the Claim Type is validated against “userlogonname” and “useridentifier”.  If the claim has neither of these attributes, then it is not recognized as valid.  If it is valid, then the entity’s profile is pulled from the UPA and the details from the profile is populated.  This is why, if your UPA isn’t functioning correctly (why isn’t it?), the details in the Entity Picker may not reflect what is in Active Directory.

Lastly, the entity is returned to the Entity Picker, and the results are presented to the user.

Interestingly enough, if you watch the Entity Picker closely enough, you’ll notice that “Organization” moves from the top of the list to the bottom of the list.  This is due to how Microsoft is first enumerating all data sources when the Entity Picker is opened, and then how the data sources are presented after the search is completed via a `StringBuilder`.