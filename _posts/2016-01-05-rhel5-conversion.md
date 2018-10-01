---
layout: post
title: Converting Rhel to CentOS
---

RHEL is a wonderful distribution, but it can be expensive to license. Red Hat has a free distribution called Fedora, but there is also a community version of the RHEL distro called CentOS. Because CentOS uses all the same Binaries as RHEL you can easily convert a RedHat box to CentOS. This is a guide for RHEL 5 to CentOS 5. 

Most of the process can be adapted to newer or older versions of the OS, you'll just need to look around for the correct files and repositories. 

I like to be lazy so I put all of this together into a script that I'll put at the end of the article. Also I'd like to give credit to the folks over at [Picky Sysadmin](https://www.pickysysadmin.ca/2014/04/27/how-to-convert-rhel-5-x-to-centos-5-x/) for inspiring this article. 

## Process

* Clean up the yum cache

    # yum clean all

* Make a working directory for the process

    # mkdir -p /temp/centos
    # cd /temp/centos

* Find the version of RHEL you're using (This will sometimes tell you the running architecture, if not use the next command)

    # more /etc/*-release

* Determine your architecture

    # uname -i

* Download the files for your release and architecture. The version numbers have likely changed or are different for your version, make sure to check.

* Download the files for release and architechure. The version numbers can change. This is written for cent 5.0, check the files at the FTP site http://mirror.centos.org/centos

    wget http://mirror.centos.org/centos/5/os/x86_64/RPM-GPG-KEY-CentOS-5
    wget http://mirror.centos.org/centos/5/os/x86_64/CentOS/centos-release-5-11.el5.centos.x86_64.rpm
    wget http://mirror.centos.org/centos/5/os/x86_64/CentOS/centos-release-notes-5.11-0.x86_64.rpm
    wget http://mirror.centos.org/centos/5/os/x86_64/CentOS/yum-3.2.22-40.el5.centos.noarch.rpm
    wget http://mirror.centos.org/centos/5/os/x86_64/CentOS/yum-updatesd-0.9-6.el5.noarch.rpm
    wget http://mirror.centos.org/centos/5/os/x86_64/CentOS/yum-fastestmirror-1.1.16-21.el5.centos.noarch.rpm

* Import the GPG key for the approperiate Verion of Centos
    rpm --import RPM-GPG-KEY-CentOS-5

* Remove the RHEL release files
    rpm -e --nodeps redhat-release

* Remove the Redhat Network plugin for yum
    rpm -e --nodeps yum-rhn-plugin

* Remove the remaining RHEL files that may be installed on the system
    rpm -e --nodeps rhn-client-tools rhn-setup rhn-check rhn-virtualization-common rhnsd redhat-logos redhat-release-notes rhel-instnum

* Install CentOS RPMs we downloaded previously
    rpm -Uvh --force /temp/centos/*.rpm

* Upgrade to CentOS from RHEL
    yum upgrade

* Once you've gotten all of this to work, you can then reboot your server and should see the CentOS graphics and title.



### Scripted Version
{% highlight bash %}
    \#/bin/bash
    
    \# Clean up yum cache
    yum clean all
    
    \# Create temporary workspace
    mkdir -p /temp/centos
    cd /temp/centos
    
    \# Download the files for release and architechure. The version numbers can change
    \# This is written for cent 5.0, check the files at the FTP site http://mirror.centos.org/centos
    wget http://mirror.centos.org/centos/5/os/x86_64/RPM-GPG-KEY-CentOS-5
    wget http://mirror.centos.org/centos/5/os/x86_64/CentOS/centos-release-5-11.el5.centos.x86_64.rpm
    wget http://mirror.centos.org/centos/5/os/x86_64/CentOS/centos-release-notes-5.11-0.x86_64.rpm
    wget http://mirror.centos.org/centos/5/os/x86_64/CentOS/yum-3.2.22-40.el5.centos.noarch.rpm
    wget http://mirror.centos.org/centos/5/os/x86_64/CentOS/yum-updatesd-0.9-6.el5.noarch.rpm
    wget http://mirror.centos.org/centos/5/os/x86_64/CentOS/yum-fastestmirror-1.1.16-21.el5.centos.noarch.rpm
    
    \# Import the GPG key for the approperiate Verion of Centos
    rpm --import RPM-GPG-KEY-CentOS-5
    
    \# Remove the RHEL release files
    rpm -e --nodeps redhat-release
    
    \# Remove the Redhat Network plugin for yum
    rpm -e --nodeps yum-rhn-plugin
    
    \# Remove the remaining RHEL files that may be installed on the system
    rpm -e --nodeps rhn-client-tools rhn-setup rhn-check rhn-virtualization-common rhnsd redhat-logos redhat-release-notes rhel-instnum
    
    \# Install CentOS RPMs we downloaded previously
    rpm -Uvh --force /temp/centos/*.rpm
    
    \# Upgrade to CentOS from RHEL
    yum upgrade
{% endhighlight %} 
