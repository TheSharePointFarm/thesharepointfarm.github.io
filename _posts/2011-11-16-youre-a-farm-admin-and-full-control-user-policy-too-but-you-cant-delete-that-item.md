---
layout: post
title: You’re a Farm Admin (and Full Control User Policy, too!). But you can’t delete that item.
tags: [sp2010]
---

So over in the [SharePoint TechNet Forums](http://social.technet.microsoft.com/Forums/en-US/category/sharepoint2010) (other MCCs, MVPs, MCMs, please get involved!), there was a thread where an Administrator who had Full Control over all of the appropriate aspects of the farm could not move an item to the Second Stage Recycle Bin, nor could he delete the item using the following PowerShell script"

```powershell
$recList = (Get-SPSite "http://spsite").RecycleBin
for ($ctr = $recList.Count-1 ; $ctr -ge 0; $ctr--) 
{
	if ($recList[$ctr].Title -match "Book1*") 
	{
		if ($recList[$ctr].ItemState -eq "FirstStageRecycleBin") 
		{
			$recList[$ctr].MoveToSecondStage()
		}
		$recList[$ctr].Delete()
	}
}
```

So here we loop through the all of the items in the First Stage Recycle Bin and if the Title of the item matches the desired filter, we move it to the Second Stage Recycle Bin and finally Delete it.  Works fine if you (the Farm Admin) delete items that you have personally deleted from the site, but not on other user’s items.  When attempting to `MoveToSecondStage()` or `Delete()`, we get the errors:

```text
Exception calling "MoveToSecondStage" with "0" argument(s): "Operation is not valid due to the current state of the object."
Exception calling "Delete" with "0" argument(s): "Operation is not valid due to the current state of the object."
```

Well, so what is the current state of the object?  If we take a look at it, the ItemState is “FirstStageRecycleBin”, exactly what we’d expect.  So why are we encountering this error?  Turns out the sproc in the Content Database is throwing us off…

First, in the `Microsoft.SharePoint.SPRecycleBinUtility` class, and I’m cutting out a whole lot of unnecessary code, we encounter this section:

```csharp
if (web == null)
{
	command.Parameters.AddWithValue("@WebId", Guid.Empty);
	command.Parameters.AddWithValue("@UserId", 0);
}
else
{
	command.Parameters.AddWithValue("@WebId", web.ID);
	command.Parameters.AddWithValue("@UserId", web.CurrentUser.ID);
}
```

Notice how if web is null, we are not passing a GUID and are setting the UserId to 0?  Well, let’s take a look at dbo.proc_MoveRecycleBinItemToSecondStage in our selected Content Database:

```sql
IF (@UserId = 0)
BEGIN
SELECT
	@WebId = R.WebId,
	@UserId = R.DeleteUserId,
	@FreedSpace = (
	SELECT 
		SUM(R2.Size)
		FROM
		TVF_RecycleBin_DelId(R.DeleteTransactionId) AS R2)
		FROM
			TVF_RecycleBin_DelIdChildId(@DeleteTransactionId, 0x) AS R
			WHERE
			R.SiteId = @SiteId AND
			R.BinId = 1
		END
	ELSE
		BEGIN
		SELECT
	@FreedSpace = (
	SELECT 
	SUM(R2.Size)
	FROM
		TVF_RecycleBin_DelId(R.DeleteTransactionId) AS R2)
	FROM
		TVF_RecycleBin_DelIdChildId(@DeleteTransactionId, 0x) AS R
	WHERE
		R.SiteId = @SiteId AND
		R.WebId = @WebId AND
		R.DeleteUserId = @UserId AND
		R.BinId = 1
	END
	IF (@@ROWCOUNT  1)
		BEGIN
		SET @Ret = 1168
		GOTO cleanup
END
```

See the difference?  If you happen to have a UserId of “0” (which as the Farm Admin, you won’t, it will be at least 1), our SELECT statement includes nothing about a UserId!  But if we do have a UserId, we pass that UserId into the SELECT statement (which is selecting items from the dbo.RecycleBin table).  When running PowerShell commands, the database context will be under the account the PowerShell window was run under, unlike via the web, where the database connection is in the context of the Application Pool account.

Since you’re not the user who deleted the item into the First Stage Recycle Bin, the SELECT statement will skip over that item and we’ll end up with error 1168 (0x490 or ERROR_NOT_FOUND), which is an accurate error.

So what is the solution?  A small modification to the PowerShell script:

```powershell
$recList = (Get-SPSite "http://sharepointsite").RecycleBin
for ($ctr = $recList.Count-1 ; $ctr -ge 0; $ctr--) 
{
	if ($recList[$ctr].Title -like "testfile*") 
	{
		if ($recList[$ctr].ItemState -eq "FirstStageRecycleBin") 
		{
			$ItemId = $recList[$ctr].Id
			$recList.MoveToSecondStage($ItemId)
			$recList.Delete($ItemId)
		}
	}
}
```

While we still loop through the items in the same fashion, we pass the ItemId back to the RecycleBin object and use the RecycleBin object methods MoveToSecondStage(item) and Delete(item) instead of using the item’s object methods.