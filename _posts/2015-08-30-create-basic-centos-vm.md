---
layout: post
title:  "Basic CentOS 6 VM Setup"
date:   2015-08-30
tags: [en]
comments: true
---

Basic steps I usually do to create a new VM server running CentOS 6. This post is to avoid myself to search the same solution again.

This post requires:

1. Installed [Oracle's VirtualBox](https://www.virtualbox.org/)
2. A minimal [CentOS 6.7 ISO image](https://www.centos.org/download/)

## Install the CentOS distribution

Open the Oracle VM VirtualBox Manager, and create New VM (type=Linux, version=Red Hat (64-bit)). Give 2GB memory, VDI. Click **Start** and point to the iso image to do the installation process.

## Setup network from guest to internet

If you did not setup network during the installation, set `eth0` interface configuration to start on boot.
{% highlight console %}
# cd /etc/sysconfig/network-scripts/
# vi ifcfg-eth0
{% endhighlight %}
Set:
{% highlight bash %}
ONBOOT=yes
{% endhighlight %}
Start it up and test access to the internet:
{% highlight console %}
# ifup eth0
# ping www.google.com
{% endhighlight %}

## Essential setup

Install the EPEL (Fedora Extra Packages for Enterprise Linux) repository. From the terminal,
{% highlight console %}
# yum install epel-release
{% endhighlight %}

Update all packages,
{% highlight console %}
# yum update
# yum groupinstall "Development tools"
{% endhighlight %}

Other useful packages
{% highlight console %}
# yum install vim
# yum install man
{% endhighlight %}

Click VirtualBox VM menu and select Devices -> Insert Guest Additions CD Image ...

In the terminal:
{% highlight console %}
# mount /dev/cdrom /cdrom
# cd /cdrom
# ./VBoxLinuxAdditions.run
{% endhighlight %}

Now you can setup a Shared Folder between host & guest VM[^1].

Reboot the system.

## Setting Desktop (optional)

If you want the desktop follow these steps:
{% highlight console %}
# yum groupinstall "X Window System"
# yum groupinstall "Desktop"
# yum groupinstall "General Purpose Desktop"
# startx
{% endhighlight %}

To enable boot on GUI, then open `/etc/inittab`, change `id:3:initdefault` into `id:5:initdefault`.

## Setup network from host to guest

By default, a new VM will have a NAT network adapter, which enables the guest VM (*our new VM*) to access the internet using the host's internet connection, but the host still cannot connect to the guest VM. This is usually enough, but if we want to create a server VM, we want to be able to connect through either browser or ssh terminal from host to the guest. The trick here is to use another network adapter and we will use `Host-only Adapter` type.

First, **shutdown the VM**.

Open **VirtualBox Preferences**, open **Network** tab, select **Host-only Networks**, add new name, say `vboxnet0`. Click OK.

Using VirtualBox Settings on the new VM, create new adapter from the Network tab settings. Select Attached To = **Host-only Adapter** and select Name = **vboxnet0**. Click Advanced setting and check **Cable Connected**.

   *before clicking OK, make a note of the MAC address*

Start again the VM, copy `ifcfg-eth0` to `ifcfg-eth1`, edit:
{% highlight bash %}
DEVICE=eth1
HWADDR=XXXX
TYPE=Ethernet
UUID=f68a4435-0c3c-4f65-9aee-28d316435528
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=dhcp
{% endhighlight %}
Write HWADDR value with the MAC address of the new adapter

{% highlight console %}
# ifup eth1
# ifconfig
{% endhighlight %}

Try to ping the new IP address from host to guest VM. If it does not work, then you need determine a static IP address for eth1 instead of dynamic IP from DHCP server.

## Finalizing

It is better to attach the new IP address from eth1 to the hostname. Edit `/etc/hosts` and add the new hostname. For example if IP address is `192.168.56.102` and hostname is `mynewserver`, then add this line
{% highlight bash %}
192.168.56.102  mynewserver
{% endhighlight %}

[^1]: If the script returns error, try reboot the system.
