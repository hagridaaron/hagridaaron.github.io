---
layout: post
title: Upgrading MySQL 5.x to 5.7 In-Place  
---

## Upgrading MySQL 5.x to 5.7

We're going to do something quite unsupported in the official documentation... Jumping many versions at a single time.

### Download
First, download the source 
~~~
wget http://dev.mysql.com/get/mysql57-community-release-el6-9.noarch.rpm
~~~

### Install
From there we can use the `localinstall` command inside yum to install local rpm packages.
~~~
yum localinstall mysql57-community-release-el6-9.noarch.rpm
~~~
We next need to stop the current mysqld service on the machine.
~~~
/etc/init.d/mysqld stop
~~~

### Upgrade
Then we can being the upgrade.
~~~
yum update mysql
~~~

There will likely be issues with your `my.cnf` configuration that need to be taken care of. *ESPECIALLY* if you're jumping multiple versions. Take the time to read through the MySQL docs and do proper testing before running this in production. 

Specifically for the 5.0 => 5.7 I experienced issues with the InnoDB engine and surrounding components. 

Read through the my.cnf and error files to see what you will need to comment out. 

You will also likely run into a ulimit error, to resolve:

~~~
ulimit -l unlimited
ulimit -n 10240 
ulimit -c unlimited
~~~

### Check Database
Startup the database in safe mode while skipping grant tables, then mysqlcheck all the tables for errors.

~~~
mysqld_safe --user=root --datadir=/var/lib/mysql --skip-grant-tables &

mysqlcheck -u root -p --auto-repair --check --all-databases
~~~

double check that the root password is working before you end the job.

~~~
mysql -u root -p
~~~

### Troubleshoot
You'll have to troubleshoot the errors that come up if mysql doesn't start. Check /var/run/mysql/[hostname].err

I found this log by doing 
~~~
locate mysql | grep err
~~~

