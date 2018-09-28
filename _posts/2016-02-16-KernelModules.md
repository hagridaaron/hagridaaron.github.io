---
layout: post
title: Working with Kernel Modules  
---
##Preface
The kernel is probably the thing you most don't want to mess up in a Linux machine (think windows registry), it just gets gross if you have to start troubleshooting it blindly. I just wanted you to know that going into this.

##Case Study
For a few months now we've had machines in our VMware infrastructure that will just fall completely off the network. At first we had no idea what was going on. We then started to see "tx_hang, txhang, tx hang" and many other variations on many of the affected servers. After many months of troubleshooting with Dell, VMware, and RedHat we've been given the task of injecting a new vmxnet3 kernel driver into one of these machines. This is just background so you understand what's going on, and why we're doing this.

##Method and Tools
In Linux you can find the modules (drivers) that are currently loaded in the kernel in */proc/modules*

There's also a nifty command that will show you the status of modules in the Kernel:

    lsmod

####Other Tools
Here are some other handy tools that we will use to get this job done. 

**depmod** - handle dependency descriptions for lodable kernel modules
**insmod** - install loadable kernel module
**lsmod** - list loaded moudles
**modinfo** -display information about a kernel module
**modprobe** - high level handling of loadable modules.
**rmmod** - unload loadable modules

####Method
It goes like this: There's currently a module called vmxnet3.ko loaded into the kernel. We need to get that thing out of there, and then inject our new module (provided by VMware, compiled by Aaron). Then we can use a few module specific tools to check the version of the loaded driver.


##Do The Work
A few directories we are going to be working with. **note**: you can find your kernel version with the following command:

    uname -r

New Module Location: *~/vmxnet-only/*
Old Module Location: */lib/modules/[kernelVersion]/kernel/drivers/net/vmxnet3/*

Make sure the permissions on the new module and old module are the same. Then copy the module file over. For this example:

    sudo cp ~/vmxnet-only/vmxnet3.ko /lib/modules/2.6.32-358.11.1.el6.x86_64/kernel/drivers/net/vmxnet3/vmxnet3.ko.new

Once we have our files in place, let's make a note of the current version of the driver. Since this is an network driver we can use the ethtool command to find out more info about the driver.

    ethtool -i eth0

which gives us:

    driver: vmxnet4
    version: 1.1.29.0-k-NAPI

####The Actual Work
Remove the current module

    rmmod vmxnet3

Make sure it's been removed

    lsmod | grep vmx

change the name

    mv vmxnet3.ko vmxnet3.ko.old
    mv vmxnet3.ko.new vmxnet3.ko

Load the module

    insmod vmxnet3.ko

Check that it loaded correctly

    lsmod | grep vmx

and lastly, check the version and make sure all that work did something useful

    ethtool -i eth0

In this case I get:

    driver: vmxnet3
    version: 1.3.9.0-NAPI

#Fin.

