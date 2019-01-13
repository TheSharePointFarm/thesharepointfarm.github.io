---
layout: post
title: SharePoint and the Web Application Proxy Role
tags: [sp2010,sp2013]
---

Windows Server 2012 R2 includes a new role, the [Web Application Proxy Role](http://technet.microsoft.com/en-us/library/dn280944.aspx). This role is meant as a replacement for such technologies as Microsoft TMG and UAG, containing some of the functionality of those products. This post will go over how to implement the basics of the Web Proxy Role.

The first requirement of the Web Proxy Role is that you must have Active Directory Federation Services in your environment. The Web Proxy Role communicates with the AD FS service endpoint, and asks for the federation service address during the configuration. In addition, the Web Proxy Role cannot reside on the same server as an AD FS instance. However, this does not mean that the site behind the Web Proxy Role is consuming SAML tokens, we can still use Classic Authentication on SharePoint!

On the Web Proxy Role server, install two NICs. One NIC will be internal facing and should contain a gateway IP address for your internal network. The external NIC will have an IP and subnet mask assigned to it, but no further TCP/IP properties are necessary. The client will be an external client. Use a hosts file to configure name resolution for test lab purposes. You'll want entries for your proxy server, the Federation Service Name, and of course the Web Application FQDN. All three of these entries should be pointing to the static IP address of the external NIC on the Web Proxy Role server.

You will need a valid certificate implemented within AD FS and a certificate available for the Web Proxy Role, as well. In this example, I have a wildcard SSL certificate for *.nauplius.local. Wildcard is clearly the easiest solution for multiple hosts under the same domain, but validate that it meets your organizational security requirements.

The last requirement for pre-authentication (not pass-through) via AD FS is to enable Constrained Kerberos Delegation for the SharePoint Web Application. This is a relatively simple process. First, record the account the Web Application's Application Pool is running as. In this case, NAUPLIUS\s-sp2013apppool. Next, set the Service Principle Names on the Application Pool account, matching the FQDN and shortname of the Web Application Alternate Access Mapping. This particular Web Application only has a single AAM, https://adfstest.nauplius.local, so my SPN configuration would look like:

```powershell
setspn.exe -A HTTP/adfstest.nauplius.local NAUPLIUS\s-sp2013apppool
setspn.exe -A HTTP/adfstest NAUPLIUS\s-sp2013apppool
```

Delegate the Web Proxy Role computer account these particular SPNs. This is done through Active Directory Users and Computers. Find the computer account and select the Delegation tab. Choose "Trust this computer for delegation to specified services only" and then choose "Use any authentication protocol". The next step is to click Add, then Users or Computers. Enter the Application Pool account (s-sp2013apppool) and find the SPN. It will be listed under a Service Type of HTTP and a User or Computer of the Web Application hostname or FQDN. When complete, the delegation will look similar to this:

![WProxy_Delegation](/assets/images/2014/02/WProxy_Delegation.png)

Enable Kerberos on the Web Application. To do this, go to Central Administration -> Manage Web Applications, highlight the configured Web Application and click Authentication Providers in the ribbon. Select the proper Zone, and then under the Integrated Windows authentication dropdown, select Negotiate (Kerberos), and click Save. Attempt to log in with Windows authentication to the Web Application from a client to validate that it works. If you're prompted for authentication 3 times or get a white screen, issue an iisreset on the SharePoint server to refresh any Kerberos tickets that have been issued.

Record the Federation Service name. This is required during the Web Proxy Role setup.

![WebProxy_FederationServiceName](/assets/images/2014/02/WebProxy_FederationServiceName.png)

In addition, create any required Relying Parties for SAML-enabled applications, such as SharePoint. I won't cover Relying Parties here, but there is an excellent AD FS end-to-end guide that continues to apply at the Share-in-dipity blog.

Deploy the Web Proxy Role from Server Manager (or PowerShell). Under Server Manager -> Manage -> Add Roles or Features, select the Remote Access Server role. Then, select the Remote Access Management Tools feature, under Remote Administration Tools. Select the type of Remote Access role to install. Select Web Application Proxy, and complete the installation.

The next step is to run the Web Application Proxy Configuration Wizard. The first thing it will ask for is the Federation Server information (AD FS). You'll notice that it asks for an administrative username and password.

![WebProxy_ConfWizard1](/assets/images/2014/02/WebProxy_ConfWizard1.png)

The next step is to provide it with a certificate, again this is using a Wildcard SSL certificate due to covering numerous hosts.

![WebProxy_ConfWizard2](/assets/images/2014/02/WebProxy_ConfWizard2.png)

The last step is simply a confirmation, including the PowerShell cmdlet you could have used.

![WebProxy_ConfWizard3](/assets/images/2014/02/WebProxy_ConfWizard3.png)

Once the wizard has completed, run the Remote Access Management Console. This console provides information on published applications as well as the status of the proxy.

![WebProxy_RemoteConsole](/assets/images/2014/02/WebProxy_RemoteConsole.png)

As you can see here, I've already published two Web Applications, a Web Application on SharePoint 2010 running in Classic mode and another on SharePoint 2013 using SAML Claims integrated with AD FS. Publishing application is extremely easy. Simply Publish a new Web Application, which involves only a couple of steps. The first step is to choose whether to use AD FS or Pass-through Authentication. Pass-through Authentication is for applications that are not SAML-enabled, such as a Classic Auth SharePoint Web Application. The only difference between choosing AD FS or Pass-through is that AD FS includes one additional step, namely selecting a pre-existing Relying Party from the AD FS service.

![WebProxy_ADFSRelyingParty](/assets/images/2014/02/WebProxy_ADFSRelyingParty.png)

The next step applies to both AD FS and Pass-Through Authentication, Publishing Settings. Here you will want to give the published application a descriptive name, and then provide the external and internal URLs. Also select a valid SSL certificate for the external URL. As you can see, I've chosen a Wildcard SSL certificate.

![WebProxy_PubSettings](/assets/images/2014/02/WebProxy_PubSettings.png)

And that is it. Like when we configured the Web Proxy Role, the confirmation screen will helpfully display the PowerShell cmdlet to perform the same function that we just performed via the GUI.

![WebProxy_Conf](/assets/images/2014/02/WebProxy_Conf.png)

The next step is to test from a client. Like most reverse proxies, the Web Proxy Role is transparent to the end user. The end user is simply directed to the Active Directory Federation Server, if using pre-authentication, or the Web Application itself, if using pass-through authentication. Since pass-through looks identical as if you were logging on locally, no screenshot is required, but here is an example of a user hitting the AD FS server via the Web Proxy Role:

![WebProxy_ADFSLogin](/assets/images/2014/02/WebProxy_ADFSLogin.png)

Those are the basics to configuring the Web Proxy Role. This role is fairly simple, but provides a powerful reverse proxy in combination with AD FS, and may provide a functional replacement to Microsoft UAG for publishing applications. If you're looking for a new reverse proxy, I would suggest taking a look at the Web Application Proxy Role to see if it fits your organizations needs.