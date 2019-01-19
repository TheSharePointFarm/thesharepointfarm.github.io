---
layout: post
title: Unable to Provision Social Features in SharePoint Server 2016
tags: [sp2016]
---

SharePoint Server 2016 appears to have a bug with the social features on the MySite host. If you go to About Me or the Newsfeed, you may see a never-ending "We're almost ready!" message. This is due to the social features not deploying when a user creates their MySite. Note that the storage feature will deploy, that is, OneDrive for Business.

When deploying a Site Collection, it picks up the features to activate list from onet.xml for that particular template. There are actually quite a few templates for a Personal Site, and Microsoft introduced one of those new templates for use on MySites by default, SPSPERS#10. The definitions can be found at C:\Program Files\Common Files\microsoft shared\Web Server Extensions\16\TEMPLATE\SiteTemplates\SPSPERS\XML\onet.xml. SPSPERS#10 is missing a certain feature to activate named "MySiteSocialDeployment" (GUID "b2741073-a92b-4836-b1d8-d5e9d73679bb"). This feature is necessary to enable the Social functionality, such as the Newsfeed.

In order to enable this functionality, we need to make it part of the SPSPERS#10 deployment. Hopefully you're not using SharePoint Server 2016 in production yet (because you're using a trial key!), but none the less, make a backup of onet.xml prior to editing it! You will likely need to replace onet.xml once a fix is released. Edit the onet.xml at the above location so it looks like this:

```xml
<Configuration ID="10" Name="FullPersonalSiteS0WithMUIAndDocumentWebParserDisabled" CustomMasterUrl="_catalogs/masterpage/mysite15.master" MasterUrl="_catalogs/masterpage/mysite15.master">
	<SiteFeatures>
		<Feature ID="0EE1129F-A2F3-41A9-9E9C-C7EE619A8C33" /> <!-- Storage deployment scenario feature -->
		<Feature ID="FA8379C9-791A-4FB0-812E-D0CFCAC809C8" /> <!-- Social data store feature -->
		<Feature ID="b2741073-a92b-4836-b1d8-d5e9d73679bb" />
	</SiteFeatures>
```

Notice I added the new Feature ID, "b2741073-a92b-4836-b1d8-d5e9d73679bb". This tells SharePoint to activate that feature when someone visits the MySite host and provisions their MySite. This will enable all of the Social functionality, including provisioning "/Lists/FeedPub", the missing list preventing the FeedIdentifier from being populated for the user's profile.

If there is a Site Master already in place for SPSPERS#10, it must be deleted and recreated. Assuming there is only a single Site Master defined, this can be done via:

```powershell
$master = Get-SPSiteMaster -ContentDatabase MySiteCDB
Remove-SPSiteMaster -SiteId $master.Id -ContentDatabase MySiteCDB
New-SPSiteMaster -Template "SPSPERS#10" -ContentDatabase MySiteCDB
```

Once the Site Master has been recreated, it should look similar to this; note the Feature ID "b2741073-a92b-4836-b1d8-d5e9d73679bb" is present in the FeaturesToActivateOnCopy property.

```powershell
$master = Get-SPSiteMaster -ContentDatabase MySiteCDB
$master.FeaturesToActivateOnCopy | ft -a

FeatureId                            WebId                                Featu
                                                                          rePro
                                                                          perti
                                                                          es
---------                            -----                                -----
0ee1129f-a2f3-41a9-9e9c-c7ee619a8c33 00000000-0000-0000-0000-000000000000
f661430e-c155-438e-a7c6-c68648f1b119 00000000-0000-0000-0000-000000000000
e9c0ff81-d821-4771-8b4c-246aa7e5e9eb 00000000-0000-0000-0000-000000000000
fa8379c9-791a-4fb0-812e-d0cfcac809c8 00000000-0000-0000-0000-000000000000
b2741073-a92b-4836-b1d8-d5e9d73679bb 00000000-0000-0000-0000-000000000000
10f73b29-5779-46b3-85a8-4817a6e9a6c2 00000000-0000-0000-0000-000000000000
73ef14b1-13a9-416b-a9b5-ececa2b0604c 00000000-0000-0000-0000-000000000000
9eabd738-48b1-4a40-a109-aa75458ed7ea 37e32102-a92f-46ea-99d3-c95ef6ce1490
```

If Site Collections were already created for users, you may have to delete their User Profile from the User Profile Service Application and delete their MySite Site Collection. Re-import the user and have them re-provision a MySite. Once the provisioning is completed, the users should be able to go to About Me or the Newsfeed and not see "We're almost ready!".

When viewing the ULS log, you may also want to filter for the Event Id "aj1lz". This will tell you what the Personal Site Capabilities (SPS-PersonalSiteCapabilities) are during deployment. For both Social and Storage scenarios, the SPS-PersonalSiteCapabilities must be "6". Storage-only is "4", which is what we would see if we did not make the above changes to onet.xml. The ULS log entries will look similar to the following.

```csharp
UserProfile.UpdatePersonalSiteCapabilities Updating value for user: DOMAIN\username value: Social, Storage
```

If those are the final values during the user's MySite site creation, you've successfully configured the change to onet.xml.

EDIT 3/18/2016: There is also an alternate way of completing the profile. If a user first creates a MySite by going to the MySite Host and waiting for OneDrive for Business to provision, then goes to the Newsfeed in the App Launcher, the profile will be provisioned correctly. If the user first goes to "About Me" with or without having provisioned a MySite, the profile will be in a 'stuck' state, where the Newsfeed will not provision.

SharePoint Server 2016 bug #1 done!