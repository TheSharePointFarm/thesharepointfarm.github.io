---
layout: post
title: 
tags: [sp2010]
---

[This post is based on the SharePoint 2016 Preview — Functionality is not guaranteed to be supported, configured identically, or exist in RTM]

SharePoint 2016 IP Access Restrictions are used to restrict access based on the client's IP address. The example below will be based on URL, as many on-premises farms do not specify Site Subscription Ids for the Site Collections.

There are two limitations for this implementation

1) The user must be using an SSL-enabled URL. Users coming over plain text will not be evaluated against the IP Access Restrictions. This makes sense based on SharePoint Online, which is SSL-only.

2) The restriction must include a wildcard, either "*" if using hostnames only in URLs, or "*.domain.tld" when using FQDNs in URLs. Unfortunately, this limits the usefulness of the IP Access Restrictions as you will be unable to block one Site Collection within a Web Application while not another.

Using the SharePoint Management Shell, use these cmdlets and exmaples.

To set the first IP Access Restriction, use:

```powershell
$wa = Get-SPWebApplication https://site.domain.tld
$site = Get-SPSite https://site.domain.tld

Set-SPIPRangeAllowList -WebApplication $wa -SiteName $site -IPV4 -IPList "172.16.0.0/16"
Set-SPIPRangeAllowList -WebApplication $wa -SiteName $site -IPV6 -IPList "0040::/48"

Set-SPIPRangeAllowList -WebApplication $wa -SiteName "*.domain.tld" -IPV4 -IPList "172.16.0.0/16" 
Set-SPIPRangeAllowList -WebApplication $wa -SiteName "*.domain.tld" -IPV6 -IPList "0040::/48"
```

Make sure to set an IPv6 Access Restriction if you have AAAA DNS records for your Web Application.

To add additional IPs or IP ranges to the IP Access Restriction, use:

```powershell
Add-SPIPRangeAllowList -WebApplication $wa -SiteName $site -IPV4 -IPList "10.10.0.0/24"
```

To enforce the IP Access Restriction, run:

```powershell
Set-SPIPAccessControlOperationMode -WebApplication $wa -OperationMode ByRequestUrl
Set-SPIPRangeAllowListSetting -WebApplication $wa -SiteName $site -IPAddressAllowanceLevel Custom
Enable-SPIPRangeAllowList -WebApplication $wa
```

The Custom value for the IPAddressAllowanceLevel refers to specifying the IPs or IP address ranges to allow to the site. The other options are BlockAll (block all traffic) or AllowAll, which will allow all traffic and ignore any IP address ranges specified.

An IP can be tested against the IP Access Restrictions by running:

```powershell
Test-SPIPRangeAllowList -WebApplication $wa -SiteName $site -IP 192.168.0.10
```

Using the above examples, the test here would return false, or no access based on the IP Access Restriction in place.

Removing an IP address range is done using the following cmdlet:

```powershell
Remove-SPIPRangeAllowList -WebApplication $wa -SiteName [-IPV6 -IP "0040::/48"]
```

If you want to disable the IP Access Restrictions entirely, run:

```powershell
Disable-SPIPRangeAllowList -WebApplication $wa
```

Close and reopen your browser in order to regain access after disabling IP Access Restrictions. When the IP Access Restrictions are enabled, and the user's client does not come from an approved IP address, they will receive an HTTP 403 Forbidden message.