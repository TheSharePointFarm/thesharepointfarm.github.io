---
layout: post
title: Nauplius.SP.UserSync Release – SharePoint Foundation User Synchronization
tags: [development,news,sp2010]
---

SharePoint Foundation does not include any mechanism to automatically synchronize User Information found in each Site Collection after the user has been added to the Site Collection.  This simple timer job iterates through each Site Collection in the Foundation farm and updates the User Information List for users present in Active Directory.

Download the solution from CodePlex [here](https://foundationsync.codeplex.com/releases/view/104661).  [Documentation](https://foundationsync.codeplex.com/documentation) is fairly sparse as there is no configuration required for this timer job.

Currently this solution is compatible with SharePoint Foundation 2010 only.