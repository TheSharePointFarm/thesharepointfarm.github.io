---
layout: post
title: SharePoint Server 2016 Hybrid Auditing (Preview) Retirement
tags: [sp2016]
---

While I was a big fan of SharePoint Server 2016 Hybrid Auditing and frequently used it sucessfully, it never left the preview status. Today, Microsoft announced that Hybrid Auditing support would be removed in the November 12th, 2019 PU. While previous logs sent to the Office 365 Security & Compliance Center will be available, no new logs will be submitted. This is documented in the Office 365 Message Center under MC190932.

This change only impacts SharePoint Server 2016. This feature was not included in SharePoint Server 2019. If you require auditing for on-premesis SharePoint, implement [Site Collection Audit Logging](https://support.office.com/article/Configure-audit-settings-for-a-site-collection-A9920C97-38C0-44F2-8BCB-4CF1E2AE22D2). Remember that Site Collection Auditing adds audit logs to the Content Database the site resides in. To manage storage, consider automatic trimming of audit logs and _not_ using audit log settings such as View which generate a significant amount of audit log entries.

> If you have configured Hybrid Auditing using the SharePoint Hybrid Configuration Wizard, please read this notification, as it affects your current audit log handling. The removal of this functionality will be facilitated through a Patch Update for SharePoint 2016 on premise, available on November 12, 2019. After installation new audit logs for Hybrid usage will not be sent to the Security & Compliance Center, though previous logs will be available. Concurrently the Security and Compliance Center will be updated to not accept Hybrid audit logs after November 12, 2019.