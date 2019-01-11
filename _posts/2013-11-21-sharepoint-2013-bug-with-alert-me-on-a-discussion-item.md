---
layout: post
title: SharePoint 2013 – Bug with Alert Me on a Discussion Item
tags: [sp2013]
---

If you attempt to set an Alert on a specific Discussion Post, like this:

[DiscussionBug1](/assets/images/2013/11/DiscussionBug1.png)

You will end up a yellow screen of death, like this:

```csharp
Value does not fall within the expected range. 
Description: An unhandled exception occurred during the execution of the current web request. Please review the stack trace for more information about the error and where it originated in the code. 

Exception Details: System.ArgumentException: Value does not fall within the expected range.

Source Error: 

An unhandled exception was generated during the execution of the current web request. Information regarding the origin and location of the exception can be identified using the exception stack trace below.  

Stack Trace: 

[ArgumentException: Value does not fall within the expected range.]
   Microsoft.SharePoint.Utilities.SPUtility.Hex(Char ch) +258
   Microsoft.SharePoint.SPContentTypeId..ctor(String id) +651
   Microsoft.SharePoint.SPViewContext.get_FolderContentTypeId() +792
   Microsoft.SharePoint.WebControls.ViewSelectorMenu.AddMenuItems() +927
   Microsoft.SharePoint.WebControls.ToolBarMenuButton.CreateChildControls() +1144
   Microsoft.SharePoint.WebControls.ViewSelectorMenu.CreateChildControls() +1427
   System.Web.UI.Control.EnsureChildControls() +189
   Microsoft.SharePoint.WebControls.TemplateBasedControl.OnLoad(EventArgs e) +99
   Microsoft.SharePoint.WebControls.ToolBarMenuButton.OnLoad(EventArgs e) +76
   System.Web.UI.Control.LoadRecursive() +116
   System.Web.UI.Control.AddedControl(Control control, Int32 index) +729
   Microsoft.SharePoint.WebPartPages.XsltListViewWebPart.CreateChildControls() +7384
   Microsoft.SharePoint.WebPartPages.WebPartMobileAdapter.CreateChildControls() +112
   System.Web.UI.Control.EnsureChildControls() +166
   System.Web.UI.Control.PreRenderRecursiveInternal() +73
   System.Web.UI.Control.PreRenderRecursiveInternal() +256
   System.Web.UI.Control.PreRenderRecursiveInternal() +256
   System.Web.UI.Control.PreRenderRecursiveInternal() +256
   System.Web.UI.Control.PreRenderRecursiveInternal() +256
   System.Web.UI.Control.PreRenderRecursiveInternal() +256
   System.Web.UI.Control.PreRenderRecursiveInternal() +256
   System.Web.UI.Page.ProcessRequestMain(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint) +4048
```

This bug appears to be 'long standing', and still exists as of the October 2013 Cumulative Update for SharePoint Server.

On the process of setting up the Alert, a process gets the Content Type ID of the Folder.

We start in the `Microsoft.SharePoint.SPViewContext.FolderContentTypeId` method, executing:

```csharp
string valueFromUrlOrViewState = this.GetValueFromUrlOrViewState("FolderCTID");
```

From there, entering GetValueFromUrlOrViewState, we enter:

```csharp
        private string GetValueFromUrlOrViewState(string name)
        {
            string str = null;
            System.Guid guid;
            NameValueCollection queryString = this.ParentContext.HttpRequestContext.Request.QueryString;
            string convertToGuid = null;
            string[] values = queryString.GetValues("View");
            if (values != null)
            {
                convertToGuid = values[0];
            }
            if ((string.IsNullOrEmpty(convertToGuid) || !SPUtility.TryParseGuid(convertToGuid, out guid)) || this.ViewId.Equals(guid))
            {
                string[] strArray2 = queryString.GetValues(name);
                if (strArray2 != null)
                {
                    str = strArray2[0];
                }
                return str;
```

Within this method, here are the key variable values:

```csharp
queryString = "{RootFolder=%2fsites%2fteam%2fLists%2fDiscussions%2fTest&FolderCTID=0x012002005959A67577638A49890C55DED9AAF2FF%2chttp%3a%2f%2fspwebapp1%2fsites%2fteam%2fLists%2fDiscussions%2fFlat.aspx%3fRootFolder%3d%2fsites%2fteam%2fLists%2fDiscussions%2fTest&FolderCTID=0x012002005959A67577638A49890C55DED9AAF2FF}"

str = "0x012002005959A67577638A49890C55DED9AAF2FF,http://spwebapp1/sites/team/Lists/Discussions/Flat.aspx?RootFolder=/sites/team/Lists/Discussions/Test"
```

From there, we come back to the Microsoft.SharePoint.SPViewContext.FolderContentTypeId method and further on down, execute:

```csharp
this.FolderContentTypeId = new SPContentTypeId(valueFromUrlOrViewState);

Where valueFromUrlOrViewState equals the variable "str".  So you want to generate a new SPContentTypeID object with a hex string and a URL.  This is where things "go wrong".

        public SPContentTypeId(string id)
        {
...
                    buffer[i] = (byte) ((byte) ((SPUtility.Hex(chArray[index]) << 4) | SPUtility.Hex(chArray[index + 1])));
...
}
```

We're calling SPUtility.Hex where chArray has invalid characters (values outside of 0 - 9 and A - F) in it.  The first one that gets hit is the "," (comma) character.  SPUtility.Hex has no return statement, so it throws a System.ArgumentException.

This particular bug will require Microsoft to resolve it.  I've opened a PSS case on it.