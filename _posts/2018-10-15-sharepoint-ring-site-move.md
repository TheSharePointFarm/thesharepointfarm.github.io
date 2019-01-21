---
layout: post
title: SharePoint Ring - Site Move!
tags: [sp2016]
---

One of the tools I maintain is [sharepointring.com](https://sharepointring.com). This tool converts various values (typically hex) into their textual representation. This allows, for example, for an administrator to determine a specific permission level a user will need based on a permissions mask check failure in the ULS log.

To get to the change for the site itself, I've moved it from Azure to GitHub... Running on [Blazor](https://blazor.net/). Blazor is C# [WebAssembly](https://webassembly.org/) (Wasm) allowing one to (more or less) run C# in a browser without having to use a web server that supports ASP.NET. This can significantly reduce costs as hosting providers who offer ASP.NET (or PHP, Python, etc.) are often more expensive than those who host plain HTML.

Blazor is experimental and "not for production" but... eh, it's a site I can play around with. Blazor, or more specifically Wasm, does not work in IE. Sorry! But it does work in Chrome, Edge, and Firefox.

The same features and functionality are still there in the site. The design is slightly different (if you haven't noticed, I'm not a web dev and am using the Blazor template :)), and that's about it from a user perspective.