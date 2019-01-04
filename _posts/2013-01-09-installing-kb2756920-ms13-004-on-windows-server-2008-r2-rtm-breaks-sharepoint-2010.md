---
layout: post
title: Installing KB2756920 (MS13-004) on Windows Server 2008 R2 RTM breaks SharePoint 2010
tags: [sp2010]
---

If you are running SharePoint 2010 on Windows Server 2008 R2 RTM, installing MS13-004 ([KB2756920](http://support.microsoft.com/kb/2756920)) will break the Security Token Service with an error similar to the below:

```text
Log Name:      Application
Source:        System.ServiceModel 3.0.0.0
Date:          1/9/2013 9:22:40 PM
Event ID:      3
Task Category: WebHost
Level:         Error
Keywords:      Classic
User:          SYSTEM
Computer:      SHAREPOINT3.nauplius.local
Description:
WebHost failed to process a request.
 Sender Information: System.ServiceModel.ServiceHostingEnvironment+HostingManager/17653682
 Exception: System.ServiceModel.ServiceActivationException: The service '/SecurityTokenServiceApplication/securitytoken.svc' cannot be activated due to an exception during compilation.  The exception message is: Method not found: 'System.String System.ServiceModel.Activation.Iis7Helper.ExtendedProtectionDotlessSpnNotEnabledThrowHelper(System.Object)'.. ---> System.MissingMethodException: Method not found: 'System.String System.ServiceModel.Activation.Iis7Helper.ExtendedProtectionDotlessSpnNotEnabledThrowHelper(System.Object)'.
   at System.ServiceModel.WasHosting.MetabaseSettingsIis7V2.WebConfigurationManagerWrapper.BuildExtendedProtectionPolicy(ExtendedProtectionTokenChecking tokenChecking, ExtendedProtectionFlags flags, List`1 spnList)
   at System.ServiceModel.WasHosting.MetabaseSettingsIis7V2.WebConfigurationManagerWrapper.GetExtendedProtectionPolicy(ConfigurationElement element)
   at System.ServiceModel.WasHosting.MetabaseSettingsIis7V2.ProcessWindowsAuthentication(String siteName, String virtualPath, HostedServiceTransportSettings& transportSettings)
   at System.ServiceModel.WasHosting.MetabaseSettingsIis7V2.CreateTransportSettings(String relativeVirtualPath)
   at System.ServiceModel.Activation.MetabaseSettingsIis.GetTransportSettings(String virtualPath)
   at System.ServiceModel.Activation.MetabaseSettingsIis.GetAuthenticationSchemes(String virtualPath)
   at System.ServiceModel.Channels.HttpChannelListener.ApplyHostedContext(VirtualPathExtension virtualPathExtension, Boolean isMetadataListener)
   at System.ServiceModel.Channels.HttpTransportBindingElement.BuildChannelListener[TChannel](BindingContext context)
   at System.ServiceModel.Channels.BindingContext.BuildInnerChannelListener[TChannel]()
   at System.ServiceModel.Channels.MessageEncodingBindingElement.InternalBuildChannelListener[TChannel](BindingContext context)
   at System.ServiceModel.Channels.BinaryMessageEncodingBindingElement.BuildChannelListener[TChannel](BindingContext context)
   at System.ServiceModel.Channels.BindingContext.BuildInnerChannelListener[TChannel]()
   at System.ServiceModel.Channels.Binding.BuildChannelListener[TChannel](Uri listenUriBaseAddress, String listenUriRelativeAddress, ListenUriMode listenUriMode, BindingParameterCollection parameters)
   at System.ServiceModel.Description.DispatcherBuilder.MaybeCreateListener(Boolean actuallyCreate, Type[] supportedChannels, Binding binding, BindingParameterCollection parameters, Uri listenUriBaseAddress, String listenUriRelativeAddress, ListenUriMode listenUriMode, ServiceThrottle throttle, IChannelListener& result, Boolean supportContextSession)
   at System.ServiceModel.Description.DispatcherBuilder.BuildChannelListener(StuffPerListenUriInfo stuff, ServiceHostBase serviceHost, Uri listenUri, ListenUriMode listenUriMode, Boolean supportContextSession, IChannelListener& result)
   at System.ServiceModel.Description.DispatcherBuilder.InitializeServiceHost(ServiceDescription description, ServiceHostBase serviceHost)
   at System.ServiceModel.ServiceHostBase.InitializeRuntime()
   at Microsoft.IdentityModel.Protocols.WSTrust.WSTrustServiceHost.InitializeRuntime()
   at System.ServiceModel.ServiceHostBase.OnOpen(TimeSpan timeout)
   at System.ServiceModel.Channels.CommunicationObject.Open(TimeSpan timeout)
   at System.ServiceModel.ServiceHostingEnvironment.HostingManager.ActivateService(String normalizedVirtualPath)
   at System.ServiceModel.ServiceHostingEnvironment.HostingManager.EnsureServiceAvailable(String normalizedVirtualPath)
   --- End of inner exception stack trace ---
   at System.ServiceModel.ServiceHostingEnvironment.HostingManager.EnsureServiceAvailable(String normalizedVirtualPath)
   at System.ServiceModel.ServiceHostingEnvironment.EnsureServiceAvailableFast(String relativeVirtualPath)
 Process Name: w3wp
```

KB2756920 removes the following two methods in `System.ServiceModel.Activation.Iis7Helper`:

```csharp
        internal static string ExtendedProtectionDotlessSpnNotEnabledThrowHelper(object obj)
        {
            return System.ServiceModel.SR.GetString("Hosting_ExtendedProtectionDotlessSpnNotEnabled", new object[] { obj });
        }

        internal static string ExtendedProtectionFlagsNotSupportThrowHelper(object obj)
        {
            return System.ServiceModel.SR.GetString("Hosting_ExtendedProtectionFlagsNotSupport", new object[] { obj });
        }
```

The above exception calls out the top missing method.  If you are running Windows Server 2008 R2 RTM, I would recommend upgrading to 2008 R2 SP1.  If that is not possible, removing KB2756920 will also resolve this particular issue, but will leave the server open to the vulnerability resolved by MS13-004.

Update: Windows Server 2008 SP2 is also not affected by this issue.

Update 2: Installing [KB2637518](http://support.microsoft.com/kb/2637518) on Windows Server 2008 R2 RTM looks to resolve the WCF issues with the Security Token Service.

Update 3: There is now a KB on this issue, [KB2801728](http://support.microsoft.com/kb/2801728).