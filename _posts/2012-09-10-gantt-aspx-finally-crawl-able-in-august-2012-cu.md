---
layout: post
title: Gantt.aspx finally crawl-able in August 2012 CU!
tags: [sp2010]
---

Previous to the August 2012 Cumulative Update, gantt.aspx would generate a browser compatibility error when it was crawled by the search crawler.  This has been resolved in the August 2012 CU by removing the browser compatibility restrictions.  The issue was that the search crawler identifies itself as Internet Explorer 4 when performing the crawl, which did not meet the previous requirement of IE7 or above for the Gantt control.

Pre-August 2012:

```csharp
protected override void OnLoad(EventArgs e)
{
	if (!this.IsBrowserSupported())
	{
		throw new SPException(SPResource.GetString("GanttBrowserNotSupported", new object[0]));
	}
	base.OnLoad(e);
	this.RegisterV4Flag();
}

private bool IsBrowserSupported()
{
	HttpBrowserCapabilities browser = this.Context.Request.Browser;
	switch (browser.Browser)
	{
		case "IE":
		return (browser.MajorVersion >= 7);
		case "Firefox":
		return (browser.MajorVersion >= 3);
		case "AppleMAC-Safari":
		return (browser.MajorVersion >= 5);
	}
return false;
}
```

Post-August 2012:

```csharp
protected override void OnLoad(EventArgs e)
{
	base.OnLoad(e);
	this.RegisterV4Flag();
}
```