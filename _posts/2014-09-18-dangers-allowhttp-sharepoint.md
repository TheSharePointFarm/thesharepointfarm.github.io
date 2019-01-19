---
layout: post
title: The Dangers of AllowHttp for SharePoint
tags: [sp2013,sp2016,sp2019]
---

SharePoint 2013 and Office Web Apps communicate authorization via a JSON Web Token. This JWT contains [important information](http://blogs.msdn.com/b/kaevans/archive/2013/04/05/inside-sharepoint-2013-oauth-context-tokens.aspx) about the caller, such as the username, SID, time-related information, as well as where the request is originating from. JSON is insecure by default and the JWT is [easily unpacked](http://openidtest.uninett.no/jwt) to reveal the contents of the JWT.

So, how does this apply to SharePoint? If you've followed what I do for any period of time, you'll probably know I'm in the SharePoint-related TechNet/MSDN forums on a daily basis. I see a lot of fun configuration issues, and so forth. But one thing I've seen on a fairly regular basis is Office Web Apps administrators setting their WAC farm to AllowHTTP = $true. This means the insecure JWT is being passed over the wire in plain text, instead of WAC's default setting of leveraging an SSL certificate for transport encryption.

To demonstrate this, two Domain User accounts are created. NAUPLIUS\User1 and NAUPLIUS\User2. User1 is granted Contribute rights on a Word Document in a SharePoint site. User2 has no access to the Word Document or the SharePoint site. When the user opens the document in Word Web App, a packet capture will be performed to capture the JWT and URI.

![WOPI3](/assets/images/2014/09/WOPI3.png)


![WOPI1](/assets/images/2014/09/WOPI1.png)

Here we can see three key pieces of information, the access token value ("eyJ0eXAiOiJKV1QiL…") and the Web the file is located on ("http://spwebapp1/sites/team"). The JWT can be unpacked to examine the contents:

```
{
    "aud": "wopi/spwebapp1@5c688b20-35bb-4fa5-b4c0-f8b66734f0b2",
    "iss": "00000003-0000-0ff1-ce00-000000000000@5c688b20-35bb-4fa5-b4c0-f8b66734f0b2",
    "nbf": "1411056774",
    "exp": "1411092774",
    "nameid": "0#.w|nauplius\\user1",
    "nii": "microsoft.sharepoint",
    "isuser": "true",
    "cachekey": "0).w|s-1-5-21-2128666325-3793459970-779680323-86357",
    "isloopback": "True",
    "appctx": "def25ded79194e09bf5e340635599a85;f+Jabf9flKXCui1eAJ5dczLct6s=;Default;;1B03C4312EF;True"
}
```

From the JWT, using the first value in appctx ("def25ded79194e09bf5e340635599a85"), it is trivial to construct a web request to view this document.

Using a custom console application that fires off the [WebBrowser](http://msdn.microsoft.com/en-us/library/system.windows.forms.webbrowser(v=vs.110).aspx) control, the above information can be inputted and viewed under any user account, regardless of their access level in SharePoint. In this example, NAUPLIUS\User2 has leveraged the JWT token from NAUPLIUS\User1 to view the previous document. And remember, NAUPLIUS\User2 has no access to the Site Collection or the Word Document.

![WOPI2](/assets/images/2014/09/WOPI2.png)

Based on this, it is technically feasible to leverage the JWT to impersonate another user. Having access to the JWT is the same as having a username and password when it comes to applications that use JWTs.

The lesson on display here is that transport encryption is vital to securing a JWT, typically in the form of SSL communications. If SSL was enabled on the WAC farm, it would not be possible to view the JWT. _Always_ use SSL in production environments, and consider it in development environments that traverse networks or contain sensitive data.