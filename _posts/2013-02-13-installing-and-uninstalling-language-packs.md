---
layout: post
title: Installing and Uninstalling Language Packs
tags: [sp2010]
---

Language Packs provide not only system-based UI elements in the language they support, but also  InfoPath forms, features, and workflows.

Installing the Language Packs is straightforward:

Install the Language Pack on all servers in the farm.  Combine it with the Language Pack's Service Pack 1 (the premise is identical to Todd Klindt's article on slipstreaming SharePoint 2010).

Run the Config Wizard after the Language Pack has been applied to all members in the farm.

To uninstall the Language Pack, it will require a few extra steps.  First, using Programs and Features, remove the target Language Pack from each server in the farm.  Once that has been completed, run the Config Wizard on each member of the farm.

If you're using SharePoint Enterprise, the Administrator-approved InfoPath forms as well as their associated Workflows (features) will be left behind.  For example, after removing the German LP, the following three features remain in an error state:

```text
3bc0c1e1-b7d5-4e82-afd7-9f7e59b60407
19f5f68e-1b92-4a02-b04d-61810ead0407
a42f749f-8633-48b7-9b22-403b40190407
```

If you navigate to a site and go to Site Settings -> Workflows, an error will be thrown indicating that a feature.xml is missing.

To remove these features, use [FeatureAdmin](http://featureadmin.codeplex.com/).  After running the application, hit the Load button, then find the features that start with "ERROR READING FEATURE".  One at a time, highlight them and click Uninstall.  Note, make sure that these are actually the Language Pack features.  The best method to do this would be to identify the Feature ID prior to uninstallation of the Language Pack.  Each Language will drop it's features in the 14 hive under `TEMPLATE\FEATURES\<featurename><LangId>`.  For example, the German LP will install a feature to `%CommonProgramFiles%\Microsoft Shared\Web server extensions\14\Template\Features\ReviewPublishingSPD1031\`.  By opening the feature.xml file, the Id of the Feature will be displayed here.

Once the dead features are removed, Site Settings -> Workflows will again function.  On SharePoint Enterprise, InfoPath Admin Approved forms will also be left behind under General Application Settings -> Manage form templates.  If you attempt to remove them from Central Administration, it will throw an error indicating that they're dependent on a feature and cannot be removed.  To remove these templates, we can use PowerShell to bind and remove them, either one-by-one or in an array:

```powershell
$xsn = Get-SPInfoPathFormTemplate "CollectSignatures_Sign_1031.xsn"
$xsn.Delete()

$xsns = (Get-SPInfoPathFormTemplate "CollectSignatures_Sign_1031.xsn"),
(Get-SPInfoPathFormTemplate "ddworkflow_assocformcode_1031.xsn"),
(Get-SPInfoPathFormTemplate "Expiration_Complete_1031.xsn"),
(Get-SPInfoPathFormTemplate "Xlate_OOB_WrapItUp_1031.xsn")
foreach($xsn in $xsns)
{
$xsn.Delete()
}
```

This will remove the target InfoPath forms from Central Administration.

A helpful list of Language IDs for SharePoint is also available here: [Language packs (SharePoint Server 2010)](http://technet.microsoft.com/en-us/library/ff463597.aspx).