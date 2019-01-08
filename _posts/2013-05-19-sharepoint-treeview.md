---
layout: post
title: SharePoint TreeView
tags: [development]
---

Part of my Word Automation Services project was to provide an alternate save location.  WAS takes this in the form of a string `url`.  I figured I would start off just by using [SPTreeView](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.webcontrols.sptreeview.aspx) control and return the selected node `url` back to the parent form.

SPTreeView, unfortunately, sets the [NavigateUrl](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.treenode.navigateurl(v=vs.90).aspx) property.  When this property is set, the [SelectedNodeChanged](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.treeview.selectednodechanged(v=vs.90).aspx) and [TreeNodeCheckChanged](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.treeview.treenodecheckchanged(v=vs.90).aspx) events do not fire.  So, instead I used a standard TreeView that is similar to the SPTreeView.

```csharp
        readonly Guid _doclib = new Guid("00bfea71-e717-4e80-aa17-d0c71b360101");

        protected void Page_Load(object sender, EventArgs e)
        {
            if (Page.IsPostBack) return;

            var webs = Web.GetSubwebsForCurrentUser();

            var rootNodeWebIcon = (string) SPUtility.MapWebToIcon(Web).First;
            var rootIcon = "/_layouts/images/" + rootNodeWebIcon;
            var rootWebNode = new TreeNode(Web.Title, Web.Url, rootIcon) {SelectAction = TreeNodeSelectAction.None};
            treeView1.Nodes.Add(rootWebNode);

            foreach (SPWeb web in webs)
            {
                var rootNode = treeView1.Nodes[0];
                var first = (string) SPUtility.MapWebToIcon(web).First;
                var icon = "/_layouts/images/" + first;
                var webNode = new TreeNode(web.Title, web.Url, icon) {SelectAction = TreeNodeSelectAction.None};
                rootNode.ChildNodes.Add(webNode);
                GetWebs(webNode, web);
                GetLibraries(webNode, web);
                web.Dispose();
            }

            GetLibraries(treeView1.Nodes[0], Web);

            treeView1.ExpandDepth = 1;
        }

        protected void treeView1_SelectedNodeChanged(object sender, EventArgs e)
        {
            Response.Write(treeView1.SelectedNode.Text);
        }

        private void GetWebs(TreeNode topNode, SPWeb rootWeb)
        {
            var webs = rootWeb.GetSubwebsForCurrentUser();

            foreach (SPWeb web in webs)
            {
                var first = (string)SPUtility.MapWebToIcon(web).First;
                var icon = "/_layouts/images/" + first;
                var webNode = new TreeNode(web.Title, web.Url, icon) { SelectAction = TreeNodeSelectAction.None };
                topNode.ChildNodes.Add(webNode);
                GetLibraries(webNode, web);
                GetWebs(topNode, web);
                web.Dispose();
            }
        }

        private void GetLibraries(TreeNode topNode, SPWeb web)
        {
            foreach (SPList list in web.Lists)
            {
                if (list.TemplateFeatureId != _doclib || list.Hidden) continue;
                var libraryTreeNode = new TreeNode(list.Title, list.RootFolder.Url, list.ImageUrl)
                    {
                        ShowCheckBox = true,
                        SelectAction = TreeNodeSelectAction.Select
                    };
                topNode.ChildNodes.Add(libraryTreeNode);
                GetFolders(libraryTreeNode, list.RootFolder);
            }      
        }

        private void GetFolders(TreeNode topNode, SPFolder rootFolder)
        {
            var query = new SPQuery {Folder = rootFolder};
            var web = rootFolder.ParentWeb;
            var listColl = web.Lists[rootFolder.ParentListId].GetItems(query);

            foreach (SPListItem listItem in listColl)
            {
                if (listItem.Folder == null) continue;
                var folderTreeNode = new TreeNode(listItem.Folder.Name, listItem.Folder.Url, "/_layouts/images/folder.gif")
                {
                    ShowCheckBox = true,
                    SelectAction = TreeNodeSelectAction.Select
                };
                topNode.ChildNodes.Add(folderTreeNode);
                GetFolders(folderTreeNode, listItem.Folder);
            }                
        }
```