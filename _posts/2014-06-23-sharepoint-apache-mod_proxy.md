---
layout: post
title: SharePoint with Apache mod_proxy
tags: [sp2010,sp2013]
---

The [Apache Software Foundation](http://apache.org/) provides a reverse proxy module named [mod_proxy](http://httpd.apache.org/docs/2.2/mod/mod_proxy.html) and [mod_ssl](http://httpd.apache.org/docs/2.2/mod/mod_ssl.html) (which extends functionality into SSL). This is a non-authenticating reverse proxy similar to function to Microsoft's IIS Application Request Routing module. This article will cover getting a single Web Application on a single SharePoint server behind the reverse proxy over SSL on port TCP/443. We will be starting from the same VM as used in the previous blog post about using [CentOS and iptables]({% post_url 2014-06-21-hyper-v-private-networks-sharepoint %}), so familiarize yourself with that before continuing as that will be the base configuration moving forward. By now I will assume that you're familiar enough with vi to know how to save files. In addition, I will assume that the Domain Controller on the Hyper-V Internal vSwitch is at 192.168.0.2 and the SharePoint server is at 192.168.0.3. In addition, our SharePoint server is going to have an SSL certificate of sharepoint.corp.nauplius.net, and of course that is what our Web Application will be. This particular SSL certificate has been issued from StartSSL (it's free).

The first step will be to modify the networking on the CentOS virtual machine.

vi /etc/sysconfig/network-scripts/ifcfg-eth0

Add a new line:

PEERDNS=no

This will disable the DHCP client (`dhclient`) from automatically adding the DHCP servers DNS information to `/etc/resolv.conf` (which is what allows the VM to automatically resolve domain names). Instead, we're going to manage that using the internal Domain Controller over eth1!

```shell
vi /etc/sysconfig/network-scripts/ifcfg-eth1
```

Add a new line:

```
DNS=192.168.0.2
```

Next, let's edit the `resolv.conf`

```shell
vi /etc/resolv.conf
```

Change it so it reads:

```
search <Active Directory FQDN>
192.168.0.2
```

The next step is to bring up and down both interfaces.

```shell
ifcfg down eth0
ifcfg up eth0
ifcfg down eth1
ifcfg up eth1
```

Once both interfaces are back up, your Windows Servers should continue to have name resolution connectivity to the Internet. In addition, `/etc/resolv.conf` should show the static settings you inputted (`cat /etc/resolv.conf`). In addition, you should be able to ping, from the CentOS VM, to the internal servers by IP and FQDN. Next, we need to install a few packages on CentOS. Apache (for `mod_proxy`), `mod_ssl` (for SSL support), and `bind_utils` (for `dig`, similar to `nslookup` on Windows).

```shell
yum install httpd
yum install mod_ssl
yum install bind_utils
```

We now need to add a couple of new firewall rules:

```shell
iptables -I INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -I INPUT -p tcp -m state --state NEW -m tcp --dport 443 -j ACCEPT
```

And save the rules:

```shell
service iptables save
service iptables restart
```

This allows iptables to accept SSL traffic to the reverse proxy (locally).

The next section will not be quite as easy. We will be dealing with SSL certificates on the CentOS VM. In order to do this properly, we'll need to upload certificates to the CentOS VM and unlike Windows, it isn't quite the point-and-click affair with PFX.

Most SSL certificate vendors will offer an unencrypted private key as well as public key file. You'll want both of these. In addition, you'll also want the appropriate Certificate Authority SSL certificate bundle (this contains the public certificate chain for your SSL certificate). If your SSL vendor does not offer these file types, you'll need to use [OpenSSL](http://www.openssl.org/) to convert the files to the appropriate file formats. For reference, here will be my file names:

Unencrypted private key: sharepoint.key

Public key: sharepoint.cer

CA bundle: startssl-bundle.pem

In addition to this, specifically for mod_ssl, we will need a single file that contains both the public and unencrypted private key in a single file. It should be in the following format:

```
-----BEGIN CERTIFICATE----- 
[Public Certificate] 
-----END CERTIFICATE----- 

-----BEGIN RSA PRIVATE KEY----- 
[Unencrypted Private Key Certificate] 
-----END RSA PRIVATE KEY-----
```

You can use a program like [Notepad++](http://notepad-plus-plus.org/) to edit the sharepoint.key and sharepoint.cer files to create the new files with the public and private key contained within it. This will be the Public-Private key bundle, and saved as `sharepointpubpriv.crt`. Now all of these files need to be transferred to the CentOS VM. In order to do this, we'll use a protocol called SCP. This protocol allows you to transfer files over SSH. Thanks to our default iptables rules, SSH is already open on our eth0 interface! Grab a copy of [WinSCP](http://winscp.net/eng/index.php) and copy these files over to a directory (e.g. /root).

The next step is to copy over the files to the appropriate default locations.

```shell
cp sharepointpubpriv.crt /etc/pki/tls/certs/sharepoint.crt
cp startssl-bundle.pem /etc/pki/tls/certs/startssl-bundle.pem
cp sharepoint.cer /etc/pki/tls/certs/sharepoint.cer
cp sharepoint.key /etc/pki/tls/private/sharepoint.key
```

Now, using vi, we need to configure `httpd.conf` (the primary Apache configuration file).

```shell
vi /etc/httpd/conf/httpd.conf
```

Make sure the following lines exist:

```
ServerName proxy.<FQDN>
KeepAlive On #Required for NTLM authentication behind the proxy!
```

In my example, I've removed the Listen 80 as well as the default VirtualDirectory. This is a reverse proxy intended to only listen on 443 and serve requests to our internal SharePoint server. There are many other settings within here that you can modify and likely should modify, but this is not an in depth lesson on Apache security.

The next step is to modify the ssl.conf file.

```shell
vi /etc/httpd/conf.d/ssl.conf
```

`LoadModule ssl_module modules/mod_ssl.so` should be present at the top of this configuration file.

Instead of providing you specific lines to set, here is the entire configuration. Again we're using my example domain here, so adjust to fit your needs.

```shell
LoadModule ssl_module modules/mod_ssl.so
Listen 443
SSLPassPhraseDialog  builtin
SSLSessionCache         shmcb:/var/cache/mod_ssl/scache(512000)
SSLSessionCacheTimeout  300
SSLMutex default
SSLRandomSeed startup file:/dev/urandom  256
SSLRandomSeed connect builtin
SSLCryptoDevice builtin
SSLStrictSNIVHostCheck off #not specifically required, but can be used with SNI in the future
##
## SSL Virtual Host Context
##

NameVirtualHost *:443
<VirtualHost *:443>
    ServerName sharepoint.corp.nauplius.net
    RequestHeader set Front-End-Https "On"
    SSLProxyEngine On 
    SSLProxyCACertificateFile /etc/pki/tls/certs/startssl-bundle.pem
    SSLProxyMachineCertificateFile /etc/pki/tls/certs/sharepoint.crt
    SSLProxyProtocol all -SSLv2
    SSLProxyCipherSuite ALL:!ADH:!EXPORT:!SSLv2:RC4+RSA:+HIGH:+MEDIUM:+LOW
    ErrorLog logs/sharepoint_ssl_error_log
    TransferLog logs/sharepoint_ssl_transfer_log
    LogLevel warn
    SSLEngine on
    SSLProtocol all -SSLv2 -SSLv3 -TLS1v1
    SSLCipherSuite ALL:!ADH:!EXPORT:!SSLv2:RC4+RSA:+HIGH:+MEDIUM:+LOW
    SSLCertificateFile /etc/pki/tls/certs/sharepoint.cer
    SSLCertificateKeyFile /etc/pki/tls/private/sharepoint.key
    SSLCACertificateFile /etc/pki/tls/certs/startssl-bundle.pem
</VirtualHost>

<Proxy *>
   Order Deny,Allow
   Allow from all
</Proxy>

ProxyRequests Off
ProxyPreserveHost On
ProxyPass / https://sharepoint.corp.nauplius.net/
ProxyPassReverse / https://sharepoint.corp.nauplius.net/
```

Make sure at the end of the `ProxyPass` and `ProxyPassReverse` lines you end them with a "/" at the end of the path, otherwise relative paths will not be returned properly to the reverse proxy and you'll see unexpected results.

Once the changes are configured, run

```shell
service httpd restart
```

Any errors will be logged in the log files at /var/log/httpd/error_log and sharepoint_ssl_error_log. To watch error_log  in real time, run:

```shell
tail -f /var/log/httpd/error_log
```

The last step would be to edit the hosts file on any client computer to target, in this case, _sharepoint.corp.nauplius.net_ at the IP of the `eth0` interface. If everything goes well, we should be prompted for credentials and let right through to the SharePoint site!