---
layout: post
title: Slow SharePoint 2013 April 2013 Cumulative Update Extraction
tags: [sp2013]
---

When running /extract against the April 2013 Cumulative Update, it takes... hours.  Using Process Monitor, it appears that the CU is going through the entire CU exe to identify and extract the files, in a one-by-one process.  For example, we'll look at the extraction of two files, edumui-es-es.msp and edumui-et-ee.msp.

First, the extraction process finds edumui-es-es.msp and creates a file as well as begins the extraction:

![Apr2013Extract1](/assets/images/2013/04/Apr2013Extract1.png)

Next, the extraction finishes, and the extraction process starts parsing the ubersrv2013 executable from the beginning again:

![Apr2013Extract2](/assets/images/2013/04/Apr2013Extract2.png)

Next, the extraction process, using 8KB to ~32KB reads, finds edumui-et-ee.msp in the ubersrv2013 executable some ways into it (about 663MB):

![Apr2013Extract3](/assets/images/2013/04/Apr2013Extract3.png)

Finally, edumui-et-ee.msp is extracted and the read process of the ubersrv2013 executable begins again:

![Apr2013Extract4](/assets/images/2013/04/Apr2013Extract4.png)

Compare to the March 2013 PU, which goes through the ubersrv2013 executable and picks out files as required:

![Mar2013PUExtract](/assets/images/2013/04/Mar2013PUExtract.png)