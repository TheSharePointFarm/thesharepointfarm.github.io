---
layout: post
title: 
tags: [sp2016]
---

The Content Type Subscriber job which is responsible for synchronizing Content Types from the Content Type Hub to Site Collections will generate unwanted audit entries regardless of the Site Collection Auditing configuration on the destination Site Collection. This bug is present in SharePoint 2010 through SharePoint 2016 as of this blog post.

This bug is due to the method `Microsoft.SharePoint.Taxonomy.ContentTypeSync.Internal.Subscriber.SynchronizeSite`.

```csharp
[...]
               using (SubscriberImport import = new SubscriberImport(site, session, termStore, proxy.IsContentTypePushdownEnabled, sharedContentTypeIds, result))
                {
                    foreach (PackageInfo info in list)
                    {
                        result.PackageId = info.PackageId;
                        try
                        {
                            if (info.IsPublished)
                            {
                                if (!string.IsNullOrEmpty(info.LocalFilePath))
                                {
                                    ULS.SendTraceTag(0x65396367, ULSCat.msoulscat_OSRV_Taxonomy, ULSTraceLevel.Verbose, "Importing package {0}", new object[] { info.LocalFilePath });
                                    import.Import(info, ref needToMarkSiteHavingExpirationPolicy);
                                    Audit(site, proxy, info.PackageId, true);
                                }
                            }
                            else if (info.PackageType == HubConstants.TypeIdContentType)
                            {
                                ULS.SendTraceTag(0x65396368, ULSCat.msoulscat_OSRV_Taxonomy, ULSTraceLevel.Verbose, "Unsharing content type {0}", new object[] { info.PackageId });
                                import.UnshareContentType(info.PackageId);
                                Audit(site, proxy, info.PackageId, false);
                            }
[...]
```

In this method, it is calling `Audit`, or `Microsoft.SharePoint.Taxonomy.ContentTypeSync.Internal.Subscriber.Audit`. In the `SynchronizeSite` method, either during the import of a package or unpublish of a Content Type to a Site Collection, the Audit method is called.

```csharp
private static void Audit(SPSite site, MetadataWebServiceApplicationProxy proxy, string packageId, bool isPublishing)
{
    StringBuilder builder = new StringBuilder();
    Uri uri = new Uri(site.Url);
    builder.AppendFormat("<ContentTypeSubscriber Uri=\"{0}\">", uri.AbsoluteUri);
    builder.AppendFormat("<Proxy>{0}</Proxy>", proxy.Name);
    builder.AppendFormat("<Package Id='{0}' Publishing='{1}' />", packageId, XmlConvert.ToString(isPublishing));
    builder.Append("</ContentTypeSubscriber>");
    string eventName = isPublishing ? Resources.GetString("AuditEventNamePackageImported") : Resources.GetString("AuditEventNameUnpublishContentType");
    site.Audit.WriteAuditEvent(eventName, Resources.GetString("AuditEventSourceSubscriber"), builder.ToString());
}
```

As it is the responsibility of the method(s) calling `SPSite.Audit.WriteAuditEvent` to evaluate the specified Site Collection Audit Setting, the above `Audit` method simply fails to check for any effective audit mask, thus writing entries to the Audit Log even if we do not have auditing enabled.

Microsoft has confirmed that this is a bug, but only for SharePoint Server 2016. While the code is nearly identical in SharePoint Server 2010 and SharePoint Server 2013, if this is fixed, it will likely only be fixed in SharePoint Server 2016 and maybe 2013. SharePoint Server 2010 is in the extended phase of support and is highly unlikely to receive a bug fix such as this. Pure speculation on my part, of course!