---
layout: post
title: There was an error during the callback and the PeoplePicker
tags: [sp2010]
---

I set up a cross-forest account (using stsadm) to search a domain for user accounts.  This worked fine for the Farm Administrator account, but did not work for the account running the application pools for other Web Applications.

On these other Web Applications, when attempting to search for "foriegn_domainuser", instead of receiving the user (or a list of matching users), I would get "There was an error during the callback".

Running Filemon while performing this, I found that the account running the application pool did not have access to the following registry key:

`HKLM\SOFTWARE\Microsoft\Shared Tools\Web Server Extensions\14.0\Secure`

Once I granted the account access, the error did not return while using the PeoplePicker to search for users.