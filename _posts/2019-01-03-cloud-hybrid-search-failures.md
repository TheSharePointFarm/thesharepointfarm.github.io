---
layout: post
title: Cloud Hybrid Search Failures
tags: [office-365,sp2013,sp2016,sp2019,spo]
---

Today, some regions may begin to notice Cloud Hybrid Search Service and SharePoint Insights failures. This is due to the SSL certificate for graph.windows.net having been renewed yesterday, 1/2/2019. When this certificate is renewed, you will begin seeing failures on the servers running the CSSA/Insights similar to the following:

```
Log Name:      Application
Source:        Microsoft-SharePoint Products-SharePoint Foundation
Date:          1/3/2019 9:39:20 AM
Event ID:      8311
Task Category: Topology
Level:         Error
Keywords:      
User:          CONTOSO\Example
Computer:      SERVER
Description:
An operation failed because the following certificate has validation errors:

Subject Name: CN=graph.windows.net
Issuer Name: CN=Microsoft IT TLS CA 1, OU=Microsoft IT, O=Microsoft Corporation, L=Redmond, S=Washington, C=US
Thumbprint: F4CE15C6CF6AD8C61CE2FFB2E1189BBD50251AAB

Errors:

NotTimeValid: A required certificate is not within its validity period when verifying against the current system clock or the timestamp in the signed file.
```

In addition, the ULS will show similar logs to the following:

```
mssearch.exe (0x2248)	0x0814	SharePoint Foundation	Topology	8311	Critical	An operation failed because the following certificate has validation errors:  Subject Name: CN=graph.windows.net Issuer Name: CN=Microsoft IT TLS CA 1, OU=Microsoft IT, O=Microsoft Corporation, L=Redmond, S=Washington, C=US Thumbprint: F4CE15C6CF6AD8C61CE2FFB2E1189BBD50251AAB  Errors:   NotTimeValid: A required certificate is not within its validity period when verifying against the current system clock or the timestamp in the signed file.
```

To resolve this, restart the SharePoint Server Search and SharePoint Search Host Controller services on all search servers to resolve Cloud Hybrid Search. If you're using SharePoint Insights, restart the SharePoint Insights service. Once restarted, the errors should disappear.

The certificate for graph.windows.net is regionally updated so not all regions will experience the issue at the same time.

You can determine when your region will expire the graph.windows.net by running the following cmdlet and using a browser to browse to the URL listed and examine the certificate.

```powershell
Get-SPTrustedSecurityTokenIssuer | ?{$_.Name -eq 'ACS_STS'} | select MetadataEndPoint
```