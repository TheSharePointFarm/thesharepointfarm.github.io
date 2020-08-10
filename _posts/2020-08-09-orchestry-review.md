---
layout: post
title: 
tags: [office-365, spo, teams]
---

Orchestry is a mix of governance and management tool. It doesn't have _reporting_ capabilities, such as ShareGate, AvePoint Cloud Governance, or a few others, but instead focuses on self-service governance -- enforcing standards and processes for provisioning sites and Teams.

Orchestry setup is quite easy, navigate to https://app.orchestry.com and follow the prompts. You will need Global Administrator rights as you will be confirming quite a few Graph API permissions. Fill in a few fields about yourself and the company, then you're good to go while the service provisions into your tenant. Orchestry will automatically provision a new Communication site at /sites/orchestry within your tenant to store assets and the site directory.

![orchestry1](/assets/images/2020/08/orchestry1.png)

The installation didn't notate this, but I didn't have an App Catalog set up in this tenant. In the Orchestry tool, post-setup there is an Installation tab under Settings which will show the status of the install. This indicated that I needed to create the App Catalog. Once I did so and waited a few minutes, I was able to provision the Orchestry-provided apps into the App Catalog with a single click. From there, Orchestry prompted me to approve Graph API permissions in the SharePoint Admin center.

![orchestry2](/assets/images/2020/08/orchestry2.png)

The next step, because I missed it during the installation process, was to import existing sites into the Orchestry Directory. The Orchestry Directory is a site directory that allows a user to view all sites in the tenant, Private, Public, and sites the end user otherwise does not have access to. Prior to import, you may want to hide the directory by going to Settings -> Teams Applications at https://app.orchestry.com. This allows you to hide particular sites from end users, in case you have sensitive sites.

Because querying the Graph is expensive, Orchestry stores all site information into a SharePoint List on a new site dedicated to Orchestry. Each site becomes a List item and has an Yes/No column "Is Visible In Directory" which when set to no, will hide that site from the Workplace Directory in Microsoft Teams. If you want to fully hide it however, you need to break List Item permissions for the entry in the Orchestry site collection. Remember that all users have read access to the Orchestry site collection!

Also present on the Orchestry site are all of the document templates that are provisioned with the templates you create. Orchestry provides a few examples out of the box.

**Site Naming**

Orchestry allows an administrator to configure Blocked Words and Naming Policies for Teams. The Naming Policies include Prefix and Suffix as well as URL naming policies. Each Naming Policy can be assigned to one or more templates. The naming and blocked word policies function like the Azure AD implementation but do not require additional Azure AD licensing.

**Templates**

Templates (SharePoint Team site, SharePoint Communications site, Microsoft Teams) can be made from scratch, an existing site/Team, or even an exported PnP template. Each template has feature sets that can be turned on, off, or be made optional. Security (Private or Public) can be enforced or the end user can pick the privacy setting. Same with document templates, and other options. This allows both administrator and end user control over provisioned sites and Teams.

Orchestry allows you to add your own features via PnP packages, such as those found on the [PnP Samples & Solutions](https://pnp.github.io/#samples) site.

***Live Templates***

Live templates are based on an existing site, but as the templated site changes, newly provisioned sites will incorporate those changes. For example, if I add a new document library to the source site, new sites provisioned from the live template will reflect that change. Sites previously provisioned based on the template will not incorporate any changes in the templated site post-provisioning.

Here's an example of creating a live template based off of an existing site. The source site has a modified home page, custom document library, and a Planner plan.

![orchestry3](/assets/images/2020/08/orchestry3.png)

In Orchestry under Workspaces -> Templates, creating a new template is easy. The first step is to fill in some basic information about the template, such as the name, template type, and any naming policies you want to enforce.

![orchestry4](/assets/images/2020/08/orchestry4.png)

If you want it to be a live template, the second step is to enable the live template feature. In addition, you can clone the Planner plan.

![orchestry5](/assets/images/2020/08/orchestry5.png)

Next, you can add features to the destination site. These come in the forms of various SharePoint Framework apps. You can enable the end user to customize the available features or enforce features. 

![orchestry6](/assets/images/2020/08/orchestry6.png)

Orchestry included example document templates for various industries, and we can add these here -- or our own document templates. Document templates are stored within the Orchestry site collection.

![orchestry7](/assets/images/2020/08/orchestry7.png)

The last configuration for a template is to allow anyone to provision a template or make the template available to only specific Active Directory groups.

**Provisioning Process**

In Microsoft Teams we can use the Orchestry app to create a new site based off of an available template.

![orchestry8](/assets/images/2020/08/orchestry8.png)

In my template I did not provide the end user with the option to override various configurations of the template, so the options are locked down.

![orchestry9](/assets/images/2020/08/orchestry9.png)

As with your standard site creation, you'll be asked for the name of the site and be provided the ability to edit the URL. Orchestry automatically fills in the Site Description, but you can adjust that as required.

![orchestry10](/assets/images/2020/08/orchestry10.png)

Once I've confirmed my selections, the provisioning process starts, cloning the existing site.

![orchestry11](/assets/images/2020/08/orchestry11.png)

Orchestry keeps a history of provisioned sites along with a significant number of details and the raw provisioning log. This can be found in Teams under the Orchestry Management tab.

![orchestry12](/assets/images/2020/08/orchestry12.png)

As with SharePoint Online sites, you can also clone Microsoft Teams. And even better, during the configuration of the template, you can remove the Wiki tab!

![orchestry16](/assets/images/2020/08/orchestry16.png)

**Metadata**

As part of the provisioning process, you can require metadata. Orchestry comes with a few pre-defined metadata, such as Department. This metadata is attached to the SharePoint List Item. This allows you to gather more information from the end user when provisioning a site or Team.

![orchestry17](/assets/images/2020/08/orchestry17.png)

![orchestry18](/assets/images/2020/08/orchestry18.png)

**Directory**

The directory is straightforward and what you'd expect -- a directory of all sites and Teams. The directory allows all users to see both Public and Private Groups as well as Communication sites even if they don't have access to the site. You can hide on a site-by-site or Team basis by going to the Orchestry site collection setting the List Item metadata "Is Visible In Directory" to No. To truly hide it, you can modify the security on the particular List Item. Using security trimming isn't ideal but your only option to hide fully hide sites from the end user (unlikely to happen, but what if you have more than 50K sites to hide?). Bulk actions may come in the future. The directory can either be displayed in a list or card view, like you see below.

![orchestry13](/assets/images/2020/08/orchestry13.png)

**Workflow**

Orchestry has a multi-step approval workflow for approving site creation. You can create workflows associated with specific templates and these workflows can have one or more approvers per step. These workflows are easy to create and associate with existing templates.

![orchestry14](/assets/images/2020/08/orchestry14.png)

**PnP Packages**

Orchestry can extract an existing site into a PnP file with the options presented by `Get-PnPProvisioningTemplate`. Once the tool has extracted the PnP file, you can view the contents of the PnP package. You can also apply the PnP package to an existing site.

![orchestry15](/assets/images/2020/08/orchestry15.png)

What I love about this tool is it brings options that would otherwise require PnP PowerShell or development skills. It has a super-simple interface and using Microsoft Teams as the end-user facing app is smart. Just build your site (webparts, Lists, Libraries, etc.) or Team (Channels, tabs, apps, etc.) and point Orchestry at it to generate a brand new template for deployment.

There are a few of rough edges I've found so far, but what I identified has either already been resolved or are minor UI issues that don't impact functionality; you will see in my screenshots a few mis-aligned elements due to the browser window size -- these don't appear in a window that is desktop sized as the Orchestry team has prioritized the desktop experience.

I think the use of a SharePoint site was both good and bad for storage of the directory and templates. The SharePoint site allows the Orchestry service to not store your data or require the costs from a self-hosted system (i.e. Azure services like a Cosmos DB for Directory and Blob Storage for templates). However, it does come with a downside of SharePoint's List limitations like List View Threshold and security trimming limit. It also makes it cumbersome to hide sites and Teams from the end user as the administrator must explicitly work with SharePoint permissions.

I would _highly_ recommend this tool to anyone looking for a self-service provisioning tool for Microsoft 365. It is quick to get set up, get your current sites into the tool, and make templates and provision new sites and Teams. The provisioning process is smart and very flexible; much better than attempting to build a PowerApp for self-service or developing your own solution. The Directory can help users find sites and Teams they're interested in or need access to without having to ask IT who the site owner is to request access.

The Orchestry team provided me with a trial tenant but otherwise didn't solicit a review. I hope this was informative and I'd like to hear your thoughts about Orchestry on [Twitter](https://twitter.com/NaupliusTrevor).
