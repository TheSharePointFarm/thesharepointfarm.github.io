---
layout: post
title: Microsoft 365 Network Connectivity Test
tags: [office-365]
---

Microsoft has introduced a preview of the Network Connectivity test for Microsoft 365. This tool measures latency, bandwidth, front door locations for Exchange Online and SharePoint Online, among other functionality from a client perspective.

Today, the OneDrive sync client sends signals to the Microsoft 365 Admin center .....

The preview service allows an end user to perform two forms of test. A simple test that measures a limited range of services can be helpful, but there is a more in depth service that requires .NET Core to be installed and local administrator rights (for the first time you run the package). This test can determine if you're running over VPN, have SSL interception through a proxy, and more. This test requires Windows and does not (today) support alternative operating systems.