---
layout: post
title: AD LDS Provider for SharePoint 2010 Foundation
tags: [ad-lds,news,sp2010]
---

SharePoint Server leverages a LDAP Membership and Role provider that is not available with SharePoint Foundation.

I've created a LdapMembership and LdapRole provider for Foundation that implements the required methods to authenticate and search for users/groups in Active Directory Lightweight Directory Services.  Make sure to leverage the same provider strings for the Web Application, SecurityTokenService, and Central Administration.

Information about installation, configuration, and uninstallation can be found on the [Downloads](http://sharepointadlds.codeplex.com/releases/view/102770) page of the project.