---
layout: post
title: SharePoint on Azure - Why I Won't Recommend It
tags: [oos,owa]
---

Microsoft is moving, or trying to get you to move everything to the cloud, either by way of Office 365 with SharePoint Online, Exchange, and other products, or via Azure for custom development and of course... Infrastructure as a Service. I've done quite a bit in IaaS. This blog used to run on a CentOS VM in Azure until I got sick of managing the VM itself :-). I routinely will set up customers to leverage Azure IaaS using ARM to run ADFS and WAP when federating with Azure AD and/or Office 365 as many of my customers don't have multiple geo-separated data centers... or perhaps don't even have one datacenter! It makes a ton of sense to make sure that WAP and ADFS are highly available. But for SharePoint on Azure? Here's why I won't recommend it. There are two primary reasons...

1) Cost
2) Licensing

Cost is an easy one. I can get a pizza box (that is, a 1 or 2U server) for near peanuts. A good one will go for somewhere around $2 - $3K if I weren't virtualizing SharePoint on an existing host. Then consider companies typically deprecate that cost over a 3 to 4 year period, and you've got yourself a very cheap server. In Azure? Just to meet the minimum requirements for SharePoint, I need a D3s v2 VM. This will cost for the VM alone (not storage or network traffic) approximately $360 USD/month (note that the price changes based on region, e.g. US West 2 is cheaper than US West). A majority of deployments I do for customers are generally 4 SharePoint servers, or roughly $1440/month. This does not include the often two SQL Servers for high availability purposes, which can range widely for cost based on requirements, or the two Domain Controllers for authentication purposes. Needless to say, just for SharePoint alone in the typical farm I work on, you'd have yourself a brand new HP or Dell pizza box every 2 months. Most companies I work with cannot see the ROI for running SharePoint in Azure.

That isn't to say it isn't for everyone -- some companies really do want to do anything they can to reduce that on-premises footprint, and Azure is a great way to do just that.

So what's this about licensing? Licensing is really easy in Azure -- for Windows, you don't have to license it at all. SharePoint, you bring your own license. But that's not the big issue. 90%+ of my customers run Office Web Apps 2013 or Office Online Server 2016 (especially true of those requiring BI functionality for SharePoint Server 2016). And thanks to Microsoft Licensing, you cannot run Office Web Apps or Office Online Server in an IaaS environment due to [License Mobility](https://azure.microsoft.com/en-us/pricing/license-mobility/); this of course extends to other IaaS provides as well, such as AWS.

Office Web Apps and Office Online Server are licensed under Office 2013 and 2016 client, respectively. You even download the server bits in the Volume Licensing Service Center under the respective Office client version. So where does License Mobility come into play? Microsoft has what is known as the [Product Use Rights](https://www.microsoft.com/en-us/Licensing/product-licensing/products.aspx), or PUR document. This document is updated generally month-to-month. Reading that document, Office Online Server is listed under subsection 2.5 underneath the Office Desktop Applications section. Section 4, Software Assurance, is what we want to look at -- namely License Mobility.

![OOSPUR](/assets/images/2016/12/OOSPUR.png)

It is this lack of right that keeps us from running Office Web Apps/Office Online Server in an IaaS environment, like Azure. While it is certainly possible to run Office Online Server in an on-premises data center while running SharePoint in Azure, it isn't generally recommended, especially when there is a heavy usage of document editing features.

I hope that Microsoft Licensing looks at re-evaluating how Office Web Apps and Office Online Server is licensed. In my personal opinion, these two pieces of software should be separated from the Office Desktop Applications and licensed on their own with License Mobility as an available option to customers. They should perhaps even be completely detached from their desktop counterparts; allow customers to purchase the server software stand alone, including editing rights.

One last note, Microsoft maintains a list of server products and features that are eligible to run within the Azure IaaS environment. That list can be found in [KB2721672](https://support.microsoft.com/en-us/kb/2721672).