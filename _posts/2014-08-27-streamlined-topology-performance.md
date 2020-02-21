---
layout: post
title: 
tags: [sp2010]
---

The [Streamlined Topology](http://www.microsoft.com/en-us/download/details.aspx?id=37000) is the 'hot new thing' with regards to SharePoint topologies. While the guidance is written for SharePoint 2013, it can be used with SharePoint 2010 as well, with the exception of the Very Low Latency tier. The explanation of this topology is that it provides better performance with low latency for end-user facing services. Let's demonstrate the Streamlined Topology performance!

This scenario should apply to most Service Applications that are designed to run on the Low Latency tier within the Streamlined Topology. Here, I'm using SSRS simply because it provides an easy-to-access render time (or, end user performance) within the Execution Log.

The key number we'll be looking at is the [TimeRendering](http://msdn.microsoft.com/en-us/library/ms159110.aspx#bkmk_executionlog3) value. This value will change depending on which server is processing the report. In this example, SP04 is the Low Latency tier -- that is, the server running the Foundation Web service that end users are accessing. The other server is SP05, or Batch tier server. While for example purposes, it also has SSRS installed, it is not the SharePoint server end users are directly interfacing with. This means that the Low Latency tier, SP04, must communicate with the Batch tier in order to gather data to render the Report.

The report created is basic. It includes a Bing Map, a Tablix, and a Matrix. All three are attached to a Data Source with data coming from a SharePoint List on the same Web Application.

With the exception of the default Service Instances and SQL Server Reporting Services, there are no other Service Instances or services running on the SharePoint servers. The virtual machines are running on the same SSD, with the SharePoint 2013 Service Pack 1 VMs allocated 8GB vRAM and 4 vCPUs, while the SQL Server is set with dynamic memory, 2GB startup and 1 vCPU. All servers are running Windows Server 2012 R2 with Update 1.

This example uses the following query against the Reporting Services database in order to collect the performance data.

```sql
SELECT InstanceName, TimeStart, TimeEnd, TimeDataRetrieval, TimeProcessing, TimeRendering
FROM ExecutionLog3
WHERE ItemPath LIKE '%CountyReport.rdl' AND Source = 'Live'
ORDER BY TimeStart DESC
```

To get a better gauge of performance, each time the Service Instance is moved from one server to another, an Iisreset is performed. In addition, the first three rendering results of each run are discarded due to JIT'ing of assemblies. The Report is not cached and does not have snapshots enabled.

The test will start with the Service Instance SQL Server Reporting Services started on SP05, the Batch Tier, and stopped on SP04, the Low Latency tier.

Using Internet Explorer 11 on Windows 8.1 Update 1 and navigating to the Report directly, the Report is refreshed with Ctrl-F5 in the browser (or, force reload). Again, the first three results are discarded, while keeping the five remaining results.

Here is the raw data for the run on SP05:

| InstanceName|TimeDataRetrieval|TimeProcessing|TimeRendering|
|-------------|-----------------|--------------|-------------|
|SP05\@Sharepoint|103|6|844|
|SP05\@Sharepoint|99|7|1055|
|SP05\@Sharepoint|99|20|1071|
|SP05\@Sharepoint|103|8|462|
|SP05\@Sharepoint|98|6|1252|

Next, the reverse test is performed, starting SSRS on SP04, the Low Latency tier, and stopping SSRS on SP05, the Batch tier.

| InstanceName|TimeDataRetrieval|TimeProcessing|TimeRendering|
|-------------|-----------------|--------------|-------------|
|SP04\@Sharepoint|118|6|710|
|SP04\@Sharepoint|102|6|632|
|SP04\@Sharepoint|115|9|421|
|SP04\@Sharepoint|98|8|566|
|SP04\@Sharepoint|102|7|543|

When SP04 is running SSRS, there is an average reduction in render times of roughly half a second, or 936.8 ms on SP05 versus 574.4 ms on SP04, a difference of 362.4 ms. While this number may not appear to be significant, the Report rendering time feels much faster with SSRS running on SP04. And how a page feels is extremely important for the end user!

This example Report used in this test is basic, but consider more complex reports that you likely have in your environment where the delta would be even higher where rendering performance is crucial.

While the Streamlined Topology model should be a serious consideration, it also requires appropriate allocation of hardware (or virtual machines). If the Low Latency tier cannot handle the load of both end users as well as the Service Instances, performance for the end user isn't going to be at an acceptable level. In this case, consider the [Traditional Topology](http://www.microsoft.com/en-us/download/details.aspx?id=30377) model instead (the classic "WFE" and "App" servers).

Another consideration, if the Business Intelligence services consume too many resources (CPU and memory), isolate them onto their own tier. While this will increase the rendering time for those Service Instances compared to a Low Latency tiered server with enough resources allocated to it, this solution is better than a Low Latency tiered server that cannot handle the Service Instance. In addition, since it is just an isolation of the Business Intelligence services, other Service Instances still benefit from the Streamlined Topology model. One important consideration to note with the Streamlined Topology model (and even the Traditional Topology) is to _dedicate_ servers to the Very Low Latency tier. This is where the Distributed Cache service will run. The Distributed Cache service is very sensitive to latency, and other services running on a Distributed Cache server can negatively impact this service, causing poor performance for end users.

With servers that have proper resource allocation, I believe the Streamlined Topology should be the topology of choice for new SharePoint farms.