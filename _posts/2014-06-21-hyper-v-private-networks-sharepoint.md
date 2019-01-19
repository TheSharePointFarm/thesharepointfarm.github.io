---
layout: post
title: Hyper-V Private Networks for SharePoint
tags: [development,sp2010,sp2013]
---

Have you ever wanted a mobile SharePoint platform, but couldn't afford Azure or other "cloud" platforms, yet could afford a 32GB laptop with a secondary SSD drive for Hyper-V? Then this post is for you!

As you know SharePoint 2013 [_requires_](http://support.microsoft.com/kb/2764086) as a Domain Controller. What does a Domain Controller require? A static IP address, of course! But what if you have a laptop, are mobile, and need to allow your SharePoint (or other servers) on Hyper-V to reach the Internet, such as to reach the public SharePoint App catalog? In order to have a static IP address, the solution is to use an Internal Virtual Switch, but in order to have Internet access, you need an External Virtual Switch. So how does one go about reconciling these needs?

Linux!

So I'll admit it. I've been using Slackware since 1997, and RedHat from roughly around that time. I used Gentoo briefly, and now use CentOS exclusively for my Linux needs. If it can do one thing well here, it can make a great proxy/firewall device for us to use within Hyper-V to route traffic. Here is what you'll need.

Create two vSwitches in Hyper-V. An External vSwitch and Internal vSwitch. Attach your Windows Servers to the Internal vSwitch and assign them to the 192.168.0.0/24 network (or any other internal network of your choosing -- it is best to not conflict with any other non-public network that may exist on the External vSwitch). It doesn't matter what IPs you assign your Windows Servers, but to keep things clear, let's reserve 192.168.0.1 for the gateway IP address.

Download CentOS-6.5-x86_64-minimal.iso from one of the [CentOS Mirrors](http://isoredirect.centos.org/centos/6/isos/x86_64/). Create a new Generation 1 VM in Hyper-V for CentOS, attach the first NIC to the External vSwitch, and the second NIC to the Internal vSwitch. I'd strongly recommend not only using this order, but creating the NICs prior to building the VM -- there is additional manual work that must be done if you add the NIC post-installation! Allocate at least 512MB RAM and 1 vCPU should be plenty for non-production use. Attach the CentOS ISO to the VM, start it up, and run the installation.

For the most part, choose the defaults. Enter a sane root password, and for partitioning, use the entire drive. The entire installation should only take a couple of minutes, and then the system will request a reboot. Note that the Microsoft Linux Integration Components are baked into the Linux kernel, so there is no need to install them.

The next step is to configure networking. The network scripts are located in `/etc/sysconfig/network-scripts/` and are named `ifcfg-eth0` (External vNIC) and `ifcfg-eth1` (Internal vNIC). Because this is a minimal install of CentOS, our text editor of choice is going to be vi. This is not the easiest text editor to use, but once you get used to it, it makes sense (I promise). I recall once being on campus at Microsoft in Redmond as part of a high school job shadow and employees argued over what editor was better vi or emacs. Clearly those who argued for emacs were wrong and probably worked on such projects as Microsoft Bob 2.0.

Let's get started.

If you ever make a mistake with vi and just want to back out of making a change, just use the following sequence of characters:

`Esc Esc : q !`

That will you out of any mode and quit without saving any changes to the file.

Ok, first thing is first, let's get an IP address from the external DHCP server on the network.

```dash
vi /etc/sysconfig/network-scripts/ifcfg-eth0
```

Use the arrow keys to go to the `ONBOOT=no` line, go to `n` in no, and hit the `Delete` key to erase `no`. Next, hit i for Insert, use the right arrow to put your cursor to the right of the = sign, and type in yes. Hit the Esc key (notice the -- INSERT -- at the bottom of the screen disappears). Next, type in :wq which means "write quit". This will save the file, and exit vi. Now, type `ifup eth0` at the prompt. This will request an IP address from the DHCP server on the network. If you were successful, you can type in ifconfig and you should see two entries, one for eth0 (our External vNIC) and another for lo (loopback). Our External vNIC will have a DHCP assigned IP address, and we should be able to ping yahoo.com (type Ctrl-C to cancel). Great! Now, onto assigning the internal IP address on eth1.

```bash
vi /etc/sysconfig/network-scripts/ifcfg-eth1
```

Using the same procedure as above, change `ONBOOT=yes`. Change `BOOTPROTO=` from `dhcp` to `none` (this means we're going to put this interface in a static IP mode). When at the end of the `BOOTPROTO=none` line, hit the Enter key, and type in the following lines:

```bash
NETWORK=192.168.0.0
IPADDR=192.168.0.1
NETMASK=255.255.255.0
```

Again, hit `Esc`, then `:wq` to save and quit `vi`. At the prompt, type `ifup eth1` which will bring the eth1 interface online.

The next step will be to enable IP Forwarding on all of our network interfaces. This is done in one of two ways, the first way is temporary until reboot, and the second way makes it permanent.

Temporary:

```bash
sysctl net.ipv4.conf.all.forwarding=1
```

Permanent:

```bash
vi /etc/sysctl.conf
```

Insert a new line at any point, adding:

```bash
net.ipv4.conf.all.forwarding=1
```

Use `:wq` to save and quit `vi`. To commit the change, from the command line, use `shutdown -r now` to restart CentOS. Log back in as root.

The next step will be to configure [iptables](http://en.wikipedia.org/wiki/Iptables). This is what will allow us to forward traffic from the internal network to the outside world. Fortunately we can do this through the iptables command line interface! Use the following commands:

```bash
iptables -A POSTROUTING -t nat -o eth0 -s 192.168.0.0/24 -d 0/0 -j MASQUERADE
iptables -I FORWARD -i eth1 -o eth0 -j ACCEPT
iptables -I FORWARD -i eth0 -o eth1 -m state --state ESTABLISHED,RELATED -j ACCEPT
```

Then, save the ruleset and restart the iptables service:

```bash
service iptables save
service iptables restart
```

The last step?

```bash
yum distribution-synchronization
```

This will update all of the packages to the latest version from the distribution's repository. You may need to restart for some of the packages to update completely. But that is it! Given your Windows Servers on the Internal vSwitch are using the IP address assigned to the Internal vNIC of the Linux VM as their Gateway, you should be able to ping to public IP addresses. Setting up a Domain Controller should also allow you to resolve public domain names, as well (given other network restrictions are not in place).

One caveat to all of this. There is an issue when moving from DHCP to DHCP network when you save the Linux VM. Unlike a Windows VM, the Linux VM will not request a new IP address from a DHCP server when coming out of a Saved State. This means you must log into your Linux VM and run dhclient to get a new client from the DHCP server on the new network you're visiting.