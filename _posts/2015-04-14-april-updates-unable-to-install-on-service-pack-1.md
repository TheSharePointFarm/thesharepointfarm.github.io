---
layout: post
title: April Updates Unable to Install on Service Pack 1
tags: [sp2013]
---

Todd has a [great article](http://toddklindt.com/WhichSP2013SP1) showing us that Service Pack 1 came in a few flavors. The first patch, with build 15.0.4569.1506, or KB2817429, was pulled. It was replaced by KB2880552, build 15.0.4571.1502. This is the version of SP1 you would be using in order to update to any future build (April 2014 CU/MS14-022 and higher).

In order to deploy on Windows Server 2012 R2, a slipstream of SharePoint 2013 SP1 was required that could only come from Microsoft. That slipstream came with build 15.0.4569.1506... the same build number as the "bad" SP1, but it was a "good" version of it. Just not the same build number as the SP1 you could download. Sort of confusing, but it has worked... Until April 2015.

The April 2015 updates have a hard requirement for Service Pack 1 to be installed for SharePoint 2013. We can see this in the XML manifest for each msp file within the downloaded executable. For example, in wdsrvloc2013-kb2965215-fullfile-x64-glb.exe ([MS15-033](https://technet.microsoft.com/en-us/library/security/ms15-033.aspx)), it contains wdsrv-x-none.msp, which is language-neutral. Along side that is wdsrv-x-none.xml, describing what build it applies to and what build it upgrades to. Previous to April, the patches targeted multiple candidates (namely, March 2013 CU and SP1). Now, they only target SP1:

```xml
<TargetVersion Validate="true" ComparisonType="Equal" ComparisonFilter="MajorMinorUpdate">15.0.4571.1502</TargetVersion>
```

But what version of SP1 is that? Yep, only the downloaded "good" version of SP1. Slipstream MSDN and VLK builds will not apply this update! Another thing we can check is the opatchinstall(n).log file that is located in %TMP% of the user running the patch. You will see lines similar to this:

```xml
OPatchInstall: Property 'WDSRV-X-NONE' value 'InvalidBaseline'
```

My suspicion is that given this impacts security updates, which have Critical and Important ratings, a fix will be produced quickly. I currently have a PSS case open for this issue, as well.

4/15/2015 Update: [Stefan Goßner](http://blogs.technet.com/b/stefan_gossner/archive/2015/04/15/common-issue-april-2015-fixes-for-sharepoint-2013-cannot-be-installed-on-sharepoint-2013-sp1-slipstream-builds.aspx) has provided a valid workaround to the issue. Apply the downloaded version of [Service Pack 1](http://www.microsoft.com/en-us/download/details.aspx?id=42544). My case with PSS is currently open and I've asked for a bug to be filed, if possible.