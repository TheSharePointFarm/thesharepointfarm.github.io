---
layout: post
title: Timer Jobs Stuck in Running Queue
tags: [sp2010,sp2013]
---

If you have one or more timer jobs stuck in the "Running" queue after removing a SharePoint server from the farm, you can either let 7 days pass and they will be removed automatically, or force the removal via PowerShell:

```powershell
$job = Get-SPTimerJob | where {$_.Name -eq "job-delete-job-history"}
$job.DaysToKeepHistory = 0 #or a value of your choosing, by default this is 7
$job.RunNow()
```

Use `$job.LastRunTime` to check the status of the job for completion.  Once the job has completed, given the timer jobs from the defunct SharePoint server are older than the DaysToKeepHistory value, the jobs will be cleaned up (along with the history for all previous valid jobs).

To reset the value for this job, run:

```powershell
$job = Get-SPTimerJob | where {$_.Name -eq "job-delete-job-history"}
$job.DaysToKeepHistory = 7
```