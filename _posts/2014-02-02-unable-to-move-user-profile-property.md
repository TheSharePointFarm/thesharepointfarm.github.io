---
layout: post
title: Unable to Move User Profile Property
tags: [sp2010,sp2013]
---

When you look at the User Profile Property list in the UPA Manage User Profiles, notice the below defaults.

![UPA_NoMove](/assets/images/2014/02/UPA_NoMove.png)

Of note is the Resource Account SID property. It is not possible to move properties above this, regardless if it is Object Exists or a custom property. When you select Move Up on a property below Resource Account SID, it will simply fail to move up. This is due to some hidden properties! Let's take a look at SQL... Here we see the property SPS-ResourceSID, which is Resource Forest SID, and SPS-ObjectExists, which is Object Exists in the Web UI. This data is from the PropertyList table in the Profile database.

![UPA_NoMoveSql1](/assets/images/2014/02/UPA_NoMoveSql1.png)

![UPA_NoMoveSql2](/assets/images/2014/02/UPA_NoMoveSql2.png)

Next, we have the DisplayOrder values. These values are how the properties are displayed in the Manage User Properties UI, with a descending DisplayOrder. Again, this is from the ProfileSubtypePropertyAttributes table in the Profile database.

![UPA_NoMoveSql3](/assets/images/2014/02/UPA_NoMoveSql3.png)

Here we see Property ID 5021 (SPS-ResourceSID) with a DisplayOrder of 5021. But what else do we see? Property ID 5022 (SPS-ResourceAccountName) should be right below it in the UI, but instead in the Manage User Profiles UI we see Property ID 5028 (SPS-ObjectExists)! So where is SPS-ResourceAccountName and how does it relate to our issue of not being able to move a property up past Resource Forest SID?

Well, there are two properties in the Manage User Profile UI that are removed from the SQL result set when the User Profile Properties are retrieved from the database, by this bit of code.

```csharp
        private void _AddPropertyToResults(DataTable dt, ProfileSubtypeProperty p)
        {
            if ((p.Name != "SPS-MasterAccountName") && (p.Name != "SPS-ResourceAccountName"))
            {
                UserProfileApplicationProxy currentApplicationProxy = ProfileAdminPage.CurrentApplicationProxy;
                using (Microsoft.Office.Server.UserProfiles.DS.getRevertedSecurityContextIfUnderWebContext())
                {
                    UserProfileConfigManager upcm = new UserProfileConfigManager(currentApplicationProxy, ProfileAdminPage.CurrentPartitionId);
                    DataRow dr = dt.NewRow();
                    this._FillNewDataRow(ref dr, p, upcm);
                    dt.Rows.Add(dr);
                    this.RowCount++;
                }
...}
```

As you can see, if the User Profile Property name is "SPS-ResourceAccountName" (or "SPS-MasterAccountName"), it is not added to the Manage User Properties list! Because of this, when you attempt to move a property above Resource Forest SID, you're actually attempting to move the property above the hidden property SPS-ResourceAccountName, which is beneath Resource Forest SID per the DisplayOrder value. When the code attached to the Up Arrow attempts to make the move for the desired property, the DisplayOder of SPS-ResourceAccountName, or 5022, is also set for the desired property. Since both properties have the same DisplayOrder, it becomes impossible to move the desired property above the hidden SPS-ResourceAccountName.

Unfortunately, there is no supported workaround for this issue.