---
layout: post
title: SharePoint Error "An error has occurred in the claim providers configured for this site collection" - Easy fix!
tags: [sp2010]
---

Well, easy if you're not using nor have configured Claims Based Authentication.  If you're not using Claims Based Authentication and you get the message "An error has occurred in the claim providers configured for this site collection.", make sure your alternate access mappings are configured correctly.  For example, if the Central Administration is at http://sharepoint:8080/, but you're using the FQDN (http://sharepoint.domain.local:8080/), you may run into this error when attempting to resolve user's name via the People Picker.  Simply add the appropriate alternate access mapping to prevent this error from occurring.