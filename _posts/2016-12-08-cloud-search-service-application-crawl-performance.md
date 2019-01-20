---
layout: post
title: Cloud Search Service Application Crawl Performance
tags: [sp2016]
---

I've been working on a Hybrid Search farm using SharePoint Server 2013 with the November 2016 Cumulative Update. The Cloud Search Service Application crawl performance was extremely poor, below 1 document per second (DPS) submitted to the SharePoint Online Search index. What I was seeing was Azure was throttling my index submissions.

```csharp
mssearch.exe (0x43B0)	0x37AC	SharePoint Server Search	Crawler:Azure Plugin	amn28	Medium	CAzurePlugin::AdviseStatus AzurePlugin is in throttling mode                    [azurepiobj.cxx:276]  search\native\gather\plugins\azurepi\azurepiobj.cxx	
mssearch.exe (0x43B0)	0x1474	SharePoint Server Search	Crawler:Azure Plugin	amn0i	High	AzureServiceProxy caught Exception: *** Microsoft.Office.Server.Search.AzureSearchService.AzureException: Throttling error while submitting document : 8726274     at Microsoft.Office.Server.Search.AzureSearchService.AzureServiceProxy.SubmitDocuments(String azureServiceLocation, String authRealm, String SPOServiceTenantID, String SearchContentService_ContentFarmId, String portalURL, String testId, String encryptionCert, Boolean allowUnencryptedSubmit, sSubmitDocument[] documents, sDocumentResult[]& results, sAzureRequestInfo& RequestInfo) ***	
mssearch.exe (0x43B0)	0x1474	SharePoint Server Search	Crawler:Azure Plugin	amn30	High	CAzurePlugin::SubmitTaskInternal Failed to SubmitDocuments with hr : 0x80040dd1  [azurepiobj.cxx:930]  search\native\gather\plugins\azurepi\azurepiobj.cxx	
mssearch.exe (0x43B0)	0x1474	SharePoint Server Search	Crawler:Common	15y3	High	>>>> Exception hr=0x80040dd1 eip=00007FFD75C15A32 module=search\native\gather\plugins\azurepi\azurepiobj.cxx line=931	
mssearch.exe (0x43B0)	0x1474	SharePoint Server Search	Crawler:Azure Plugin	amn3o	High	CAzurePlugin::SubmitTask caught NLBaseException AzureException Throttling error while submitting document : 8726274 hr=80040DD1  [azurepiobj.cxx:701]  search\native\gather\plugins\azurepi\azurepiobj.cxx	
```

This particular Content Source had roughly 9 million items to index, and due to these throttling events, the indexing was taking significantly longer than I had hoped for; to give you an idea, roughly 550k items in 150 hours. Eventually, a support case with Microsoft was opened via the Office 365 tenant. That support told me because it is hybrid, I needed to have a standard support case opened via Product Support Services. This is despite the option to pick "hybrid search" as a category in the Office 365 tenant support ticket creation.

Back on-prem, support had me implement a new property, "EnableNoGetStatusFlight", on the Cloud SSA.

```powershell
$ssa = Get-SPEnterpriseSearchServiceApplication
$ssa.SetProperty("EnableNoGetStatusFlight",1)
$ssa.Update()
```

Once the property is in place, you must restart the OSearch15 service on servers hosting the Crawl component. There is no other documentation on this property yet, but it prevents the crawler from requesting the status of submitted batches to SharePoint Online, thus increasing throughput.

After implementing this property, roughly 1 million items were crawled in the span of about 14 hours. That's a significant improvement to our DPS.

This isn't a cure-all and you still have a chance to be throttled as it depends on the rate in which you are submitting items to Azure as well as what load the SharePoint Online search farm is under.