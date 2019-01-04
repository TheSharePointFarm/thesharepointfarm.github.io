---
layout: post
title: The User Profile Synchronization Log Timestamps
tags: [sp2010]
---

I’ve seen this question asked a few times now: why does the User Profile Synchronization Status log record times in UTC and not the Windows Server, or SharePoint Server specified time zones?  The answer is that Microsoft decided to use UTC internally on this log.  Prior to SharePoint 2010 SP1, the local SharePoint Server’s time was passed into this function and the log showed the server’s time as well:

```csharp
internal SynchronizationRunStatus(SynchronizationStage stage, string connName, string runProfileName, DateTime beginTime, DateTime endTime, int adds, int nochanges, int updates, int failures, string lastFailure, SynchronizationState state, int runNumber)
{
	this.stage = (int) stage;
	this.connName = connName;
	this.runProfileName = runProfileName;
	this.beginTime = beginTime;
	this.endTime = endTime;
	this.adds = adds;
	this.updates = updates;
	this.nochanges = nochanges;
	this.failures = failures;
	this.lastFailure = lastFailure;
	this.state = (int) state;
	this.runNumber = runNumber;
}
```

Post-Service Pack 1, this function changed and was made a tad neater, but leveraged UTC instead of the local server’s time.  This is why the log shows UTC instead of the local time zone.

```csharp
internal SynchronizationRunStatus(SynchronizationStage stage, string connName)
{
	this.stage = (int) stage;
	this.connName = connName;
	this.beginTime = DateTime.UtcNow;
	this.endTime = DateTime.MaxValue;
	this.adds = this.updates = this.deletes = this.nochanges = this.failures = -1;
	this.state = 1;
}
```

And of course, the log itself showing the start time on the User Profile Service Application page at 1AM PST, but running around 8AM UTC:

![image2](/assets/images/2012/09/image2.png)