---
layout: post
title: Enabling HTTP Strict Transport Security for SharePoint Server 2016
tags: [sp2016]
---

SharePoint Server 2016 supports enabling HTTP Strict Transport Security (HSTS) to each individual Web Application. HSTS provides the following key features:

* Automatically convert any http:// (HTTP) links to https:// (SSL)
* In the future, only connect to this site via SSL
* If the connection to the site cannot be secured, do not allow access to the site

The last part is very important -- it helps prevent Man-in-the-Middle attacks. MitM attacks could come from DNS poisoning (telling the client that example.com lives at the attacker's IP address rather than the real IP address) or from corporate SSL decryption techniques, where corporations intercept and decrypt outbound-SSL traffic from their clients prior to re-encrypting it to the destination.

These are a boon for privacy and protection. To enable this feature on a SharePoint Web Application, we can use the SharePoint Management Shell. You will need to get the Web Application and then set the `HttpStrictTransportSecuritySettings` property.

```powershell
$wa = Get-SPWebApplication https://sharepoint.example.com
$wa.HttpStrictTransportSecuritySettings.IsEnabled = $true
$wa.Update()
```

There is also a `MaxAge` attribute for the `HttpStrictTransportSecuritySettings` property. This value is 31536000 represented in seconds, by default. 31536000 seconds is 365 days. The value can be raised or lowered, but what it does is tell clients to then connect to the Web Application over SSL for that period of time, with no attempts to connect over HTTP. This value should represent how long you expect to maintain valid SSL support on the Web Application; that is, not reverting back to HTTP or letting the certificate expire. MaxAge can be set via PowerShell with a maximum value of 2147483647, or ~68 years. Consider using a value between 18 weeks (10886400) and the default value of 1 year.

```powershell
$wa = Get-SPWebApplication https://sharepoint.example.com
$wa.HttpStrictTransportSecuritySettings.MaxAge = 10886400
$wa.Update()
```

This will enable HSTS and set the MaxAge value to 18 weeks. Using a browser, we can then see the response header from the Web Application shows us that HSTS is enabled.

![HSTSEnabled](/assets/images/2016/04/HSTSEnabled.png)

Note the `Strict-Transport-Security` response header, telling our browser that the server supports HSTS.

To learn more about HSTS, visit the [Wikipedia page](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security) which goes over the concepts of HSTS along with browser support. All browsers supported by SharePoint Server 2016 support HSTS.