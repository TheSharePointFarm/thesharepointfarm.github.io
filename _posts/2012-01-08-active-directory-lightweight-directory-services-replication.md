---
layout: post
title: Active Directory Lightweight Directory Services – Replication
tags: [ad-lds, sp2010]
---

Replicating an AD LDS instance is quite simple.  First, install the AD LDS role on the second server.  Next, launch the Active Directory Lightweight Directory Services Setup Wizard.  Select “A replica of an existing instance”.

![image3](/assets/images/2012/01/image3.png)

Name the instance appropriately and select the appropriate LDAP and LDAPS ports.  Enter the FQDN of the first AD LDS server, and port that the AD LDS instance is listening on.  The LDAP port has a restriction of either TCP/389 or a TCP port between 1025 and 65535.

![image11](/assets/images/2012/01/image11.png)

Enter an account that has Administrative credentials on the instance (e.g. a Local Administrator).  Select the Application Partition(s) to replicate.

![image15](/assets/images/2012/01/image15.png)

Note that as future Application Partitions are added to the AD LDS instance, they will automatically be replicated.

Select the file location for the AD LDS instance as well as what account the Windows Service will run as.  Choose an Instance administrator.  Finally, the AD LDS instance will be set up and begin replication.

![image19](/assets/images/2012/01/image19.png)

If LDAPS (SSL) is to be used on the server hosting the replicated instance, a certificate must be created and SSL set up just like the primary server hosting AD LDS.