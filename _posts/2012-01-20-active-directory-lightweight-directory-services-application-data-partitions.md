---
layout: post
title: Active Directory Lightweight Directory Services – Application Data Partitions
tags: [sp2010]
---

An AD LDS instance can hold more than one Application Data Partition.  The Application Data Partition is where user, group, etc. objects are stored.  It can provide an effective boundary between partitions, and is useful for SharePoint when using a single AD LDS instance with multiple customers that must remain isolated from each other.

One restriction with Application Data Partitions is that one partition cannot inherit the path of another partition.  E.g., you cannot create a partition of “CN=SharePoint,DC=fabrikam,DC=local” if you already have a partition of “DC=fabrikam,DC=local”.  You can, however, create multiple partitions based off of the same root DN, e.g. “CN=SharePoint,DC=fabrikam,DC=local” and “CN=SharePoint2,DC=fabrikam,DC=local”.

To create an Application Data Partition, first connect and bind with an authenticated user to the AD LDS instance with ldp.exe.  Note that this instance already has a partition of “CN=SharePoint,DC=fabrikam,DC=local”:

![image3](/assets/images/2012/01/image3.png)

In ldp.exe, go to Browse –> Add Child.

Enter the new DN, e.g. “CN=SharePoint2,DC=fabrikam,DC=local”.  Next, under Edit Entry, set the Attribute to “objectClass” and Values to “container”, then click Enter.  Next, change the Attribute to “instanceType” and the Value to “5”, again click Enter.  It should look similar to this:

![image7](/assets/images/2012/01/image7.png)

Next, click Run.  Message text similar to this will appear in the log window:

```text
-----------
***Calling Add...
ldap_add_s(ld, "CN=SharePoint2,DC=fabrikam,DC=local", [2] attrs)
Added {CN=SharePoint2,DC=fabrikam,DC=local}.
-----------
```

The new partition has been created.  Next, connect to the new partition with ADSI Edit, using the same server/port combination, but the new DN.

If you receive the following error when attempting to connect to the partition with ADSI Edit, close ADSI Edit and re-open it, then try to connect again.

![image14](/assets/images/2012/01/image14.png)

To delete an Application Data Partition, first run ADSI Edit.  Connect to the Configuration partition to identify the Configuration DN.

![image18](/assets/images/2012/01/image18.png)

Next, run ldp.exe.  Connect and bind as an authenticated user.  Go to View –> Tree, and connect to the Configuration DN of the AD LDS instance.  Find “CN=Partitions” and expand the tree.  Each Application Data Partition will have a DN starting with “CN={GUID}”  Click on one of the containers to show the information about the container.

![image30](/assets/images/2012/01/image30.png)

Find the attribute “nCName”.  This container corresponds to the “CN=SharePoint2,DC=fabrikam,DC=local” Application Data Partition.  To delete it and all data within the partition.  Ldp should output a log similar to the following:

```text
-----------
ldap_delete_s(ld, "CN=7a1c0eda-9e1c-4307-9b4b-2475c8262a6f,CN=Partitions,CN=Configuration,CN={51435092-6E63-4B14-B4CA-F27C35BE886F}");
Deleted "CN=7a1c0eda-9e1c-4307-9b4b-2475c8262a6f,CN=Partitions,CN=Configuration,CN={51435092-6E63-4B14-B4CA-F27C35BE886F}"
-----------
```

Attempting to connect to the instance from AD LDS will now yield an error similar to this:

![image34](/assets/images/2012/01/image34.png)

The partition has been successfully deleted.