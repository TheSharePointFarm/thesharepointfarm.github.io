---
layout: post
title: Active Directory Lightweight Directory Services – Recycle Bin
tags: [ad-lds, sp2010]
---

To enable the AD LDS Recycle Bin, open a command prompt, navigate to C:\Windows\ADAM, and run:

```text
ldifde.exe -i -f MS-ADAM-Upgrade-2.LDF -s localhost:389 -j . -$ adamschema.cat
```

Where “localhost:389” is the host and port where AD LDS is running.  If the currently logged on user does not have access to the AD LDS instance, specify the –b switch with a username and –p switch with a  valid password.  Note that before running this command, all members involved in AD LDS instance replication should be online.

The output should look like this:

```text
Verifying file signature
Connecting to "localhost:389"
Logging in as current user using SSPI
Importing directory from file "MS-ADAM-Upgrade-2.LDF"
Loading entries...............................
30 entries modified successfully.

The command has completed successfully
```

Now, delete a user via ADSI Edit (or any other method).  Next, from an elevated command prompt, run ldp.exe.  Go to Options –> Controls and select Return deleted objects from the Load Predefined drop down menu and click OK.

![image19](/assets/images/2012/01/image19.png)

Next, under Connection, Connect, then Bind as an Administrative user to the AD LDS instance.  Finally, under the View menu, show the Tree of the Domain BaseDN.  There is a new container for Deleted Objects:

![image10](/assets/images/2012/01/image10.png)

Right click a Deleted object.  Add the Attribute “isDeleted” and under Operation select “Delete”, then click on Enter.  Next, add the Attribute “distinguishedName” and enter the full DN to where the object should be recovered to, change the Operation to “Replace” and click on Enter.  Select the Extended check box.  It should look similar to this:

![image14](/assets/images/2012/01/image14.png)

Click Run.  The object will then be moved to the desired location and removed from the Deleted Objects container.

![image18](/assets/images/2012/01/image18.png)

Finally, in ADSI Edit, reset the user’s password and set the msDS-UserAccountDisabled attribute to False.  Set the attribute used for the user’s login (e.g. mail).  Once completed, the user will be able to log back into SharePoint.