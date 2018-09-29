---
layout: post
title: CentOS virtual networking, virbr and vlan tagging
---

# VLAN Tagging
VLANS are great for security, and are really helpful if you want to put together anything more than a simple network. Although VLAN technlolgy can be a great value add to your network, it comes with a fair bit of complexity. Get ready to put your thinking caps on, and have a whiteboard near by!

This won't be a comprehensive discussion about VLAN's, VLAN Tagging, or VLAN Trunking, but rather a little introduction to VLANs in CentOS networking. 

## Basic Network Configuration
First of all we're going to want to check out our networking config and see what we're working with. 

    ip a

This will give you a list of ip addresses and devices on your machine. Take a look and make note of your main device, there may be several depending on your hardware/software configruation. 

Next navigate over to the network-scripts directory located at */etc/sysconfig/network-scripts*

If you list out the contents of the directory, you will see a number of scripts and configuration files for different interfaces for the machine. 

    ls -lash

Next we need to make sure that we have the correct 8021q modlue loaded, this is usually loaded by default but it never hurts to check.

    modprobe --first-time 8021q
    modprobe: ERROR: could not insert '8021q': Module already in kernel

If you'd like more information about the module, use the modinfo command:

    modinfo 8021q

## Network Interface Configuration with ifcfg files

Next we're going to setup our network interfaces to use vlan tagging using ifcfg files. 

Configure the parent interface in /etc/sysconfig/network-scripts/ifcfg-ethX, where X is a unique number corresponding to a specific interface, as follows:


    DEVICE=ethX
    TYPE=Ethernet
    BOOTPROTO=none
    ONBOOT=yes


Configure the VLAN interface configuration in the */etc/sysconfig/network-scripts/* directory. The configuration file name should be the parent interface plus a . character plus the VLAN ID number. For example, if the VLAN ID is 192, and the parent interface is eth0, then the configuration file name should be *ifcfg-eth0.192*:
    
    DEVICE=ethX.192
    BOOTPROTO=none
    ONBOOT=yes
    IPADDR=192.168.1.1
    PREFIX=24
    NETWORK=192.168.1.0
    VLAN=yes

If there is a need to configure a second VLAN, with for example, VLAN ID 193, on the same interface, eth0, add a new file with the name eth0.193 with the VLAN configuration details.

Restart the networking service in order for the changes to take effect. As root issue the following command:

    # systemctl restart network

# Virtual Bridging 

I've been working on setting up a KVM server and part of that config requires a network bridge to route connections between the virtual machines that will live inside that server. Setting up a bridge is fairly straightforward once you get your head around the config files.

We will keep our orignial physical interface as is but will assign its IP-address to the bridge. We will continue with the ethX example, without the VLAN tagging. The example config would look something like this:

    DEVICE=ethX
    ONBOOT=yes
    IPADDR=192.168.1.1
    PREFIX=24
    NETWORK=192.168.1.0
    BOOTPROTO=static

The first thing we want to do is comment out everything related to an IP address and tell the interface that it will be using a bridge as well as where that bridge is located.

    DEVICE=ethX.192
    ONBOOT=yes
    #IPADDR=192.168.1.1
    #PREFIX=24
    #NETWORK=192.168.1.0
    #BOOTPROTO=static
    BRIDGE=virbr0

Next we can fcreate the config-script for the bridge interface birbr0 in /.etc/sysconfig/network-scripts/ifcfg-virbr0. Most details can be copied from the orignial script for ethX:

    DEVICE=virbr0
    TYPE=BRIDGE
    ONBOOT=yes
    IPADDR=192.168.1.1
    PREFIX=24
    NETWORK=192.168.1.0
    BOOTPROTO=static
    DNS1=192.168.1.2

After you're done with this configuration file you're basically done. Run the network restart command via systemctl:

    systemctl restart network 
    

# Putting it together
Now that we've seen how to do all of the above, we want to put everything together. 

You will create your ethX, ethX.192, and virbr0 config-scripts, but this time you'll want to comment out the IP information from the ethX.192 file and and put it into the ifcfg-virbr0 config. This will make more sense when you look at the configuration.

**ifcfg-ethX**
    
    DEVICE=ethX
    TYPE=Ethernet
    BOOTPROTO=none
    ONBOOT=yes
    
**ifcfg-ethX.192

    DEVICE=em1.27
    BOOTPROTO=none
    ONBOOT=yes
    VLAN=yes
    BRIDGE=virbr0

**ifcfg-virbr0**
    
    DEVICE="virbr0"
    TYPE=BRIDGE
    ONBOOT=yes
    BOOTPROTO=static
    IPADDR="192.168.1.1"
    PREFIX=24
    NETWORK=192.168.1.0
    VLAN=yes
    DNS1=192.168.1.2
