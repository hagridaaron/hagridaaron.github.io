---
layout: post
title: Installing DB2 Express-C Database Server 
---

In the 70's IBM started research and development on relational databases. Out of that R&D came the DB2 family of Databases. We will focus on installing the community (free) version of the family. 

##Prerequisites

* Centos >= 6 installed
* C++ compatibility libraries

    yum install compat-libstdc++-33
    yum install libaio

##Install DB2

You can find the download for DB2 Express-C [Here](http://www-01.ibm.com/software/data/db2/express-c/download.html).

login as root or use sudo, unzip and install:

    tar zxvf v10.5_linuxx64_expc.tar.gz
    cd expc
    ./db2_install

##Post installation tasks

Create groups for db2 use, these groups are for different aspects of the databae users, db2grp1 for users, db2fgrp1 for fenced users, and dasadm1 for db2 admin server administrators.


    groupadd db2grp1
    groupadd db2fgrp1
    groupadd dasadm1

Now we will create users for the groups we just created as well as set their group membership and home directories. 

    useradd -g db2grp1 -m -d /opt/ibm/db2/V10.5/db2inst1 db2inst1
    useradd -g db2fgrp1 -m -d /opt/ibm/db2/V10.5/db2fenc1 db2fenc1
    useradd -g dasadm1 -m -d /opt/ibm/db2/V10.5/dasusr1 dasusr1

Make sure to change the password and make note of it in your password manager (LastPass)

    passwd db2inst1
    passwd db2fenc1
    passwd dasusr1

At this point you can continue down the path of CLI, or you can install X and use the GUI, the choice is yours. 
[THIS](http://www-01.ibm.com/support/knowledgecenter/SSEPGG_10.5.0/com.ibm.db2.luw.qb.server.doc/doc/c0008711.html?lang=en) link can give you a bit more information about the nitty gritty details. 
