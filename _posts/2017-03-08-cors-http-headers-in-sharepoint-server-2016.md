---
layout: post
title: CORS HTTP Headers in SharePoint Server 2016
tags: [sp2016]
---

Cross Origin Resource Sharing (CORS) HTTP header values in SharePoint Server 2016 has been hard coded by the SharePoint Product Group.

Since SharePoint only accepts OAuth for CORS requests, and not user authentication such as cookies, Cross Site Request Forgery is a non-issue as origin validation does not need to take place when using OAuth. As such, setting `Allow-Cross-Origin-Request` to `*` becomes a non-issue.

Here is the code used by SharePoint. Notice you can track the addition of these headers in the ULS log.

```
ULS.SendTraceTag(0x65535a, ULSCat.msoulscat_WSS_Csom, ULSTraceLevel.Medium, "Adding response headers to CORS request.");
httpResponse.Headers.Set("Access-Control-Allow-Origin", "*");
httpResponse.Headers.Set("Access-Control-Max-Age", "2592000");
```

This also means you do not need to modify the web.config, and of course adding `Access-Control-Allow-Origin` or `Access-Control-Max-Age` will be _ignored_ by SharePoint if added to the web.config.

This may prevent certain scenarios (such as using user credentials) which did function in previous versions of SharePoint from working in SharePoint Server 2016.