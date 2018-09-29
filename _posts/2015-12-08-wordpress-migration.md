---
layout: post
title: Moving Out? Take Wordpress With You!
---

## A Guide On Moving WordPress Blogs From Host To Host

Everyone loves the usibility and abundance of wordpress utilities out in the world. This means that everywhere you look you see wordpress blogs. This is wonderful! ...until you have to move hosting providers, change servers, or want to move from dev/test to prod. This guide aims to give you the tools to move your WordPress instance from one machine to the other. 

There are a few steps:

* Install Wordpress on New Server
* Export Database from Old Host
* Import Database to New Host
* Transfer WP config files from Old Host to New Host

For this example we will use **server1** (old host) and **server2** (new host)

### Install Wordpress on New Server

Can use the instructions on the Wordpress site, or check out the ansible playbook I wrote to automate the process [Link to Github](https://github.com/hagridaaron/ansible-wordpress).


### Export Database From Old Host

We need to make sure that our new site will work at the new url. To do this, go into wordpress and select *Settings* from there you can edit the *WordPress Address* and *Site Address*.

There are a few ways to do the export, as listed in the ["Backing up your wordpress database"](https://codex.wordpress.org/Backing_Up_Your_Database#Backing_Up_the_Database) guide on the WordPress site. For this example I will be using the MySQL Workbench Method. I found this easier than setting up phpMyAdmin or running the commands on the servers themselves. This method isn't better than any of the others, it's just what I chose. 

Make sure you have MySQL-Workbench installed on your system. If not, you can use your local package manager or google to get this installed.

**Centos**

{% highlight bash lineos %}
    yum install mysql-workbench
{%  endhighlight %}

**Ubuntu**

{% highlight bash lineos %}
    apt-get install mysql-workbench
{%  endhighlight %}

**Arch**

{% highlight bash lineos %}
    pacman -S mysql-workbench
{%  endhighlight %}

**Mac OS X**

{% highlight bash lineos %}
    brew install mysql-workbench
{%  endhighlight %}

Once you have the workbench installed open it up and select *Databse > Connect to database..*

You then can select your connection method. I chose to go with SSH because we don't have many ports open between zones on our network. 

Proceed to enter in your Connection Parameters (Usernames/passwords for hosts and databases) don't worry about the default schema. 


    SSH Hostname: server1.domain.com
    SSH Username: atracy
    SSH Password: ************************
    MySQL Hostname: 127.0.0.1 (because we're logging in with ssh)
    MySQL Server Port: 3306
    Username: root
    Password: ************
    
On the left hand side, select the button that says *Data Export*

Select the tables to export:
    
    wordpress
    semi

We will then select *Dump Stored Procedures and Functions*, *Dump Events*, and *Dump Triggers* under **Objects to export** just to be safe.

Next we can select a directory on our local machine that the workbench will dump our Project Folder. For this example

    /home/atracy/dumps/Dump20141208

Close the connection to the databse, and we will move on to the next step.

### Import Database To New Host

We will do something very similar but in reverse order. With one or two added steps.

Login to server2 using the same steps from before, changing usernames and passwords where need be.

    SSH Hostname: server2.domain.com
    SSH Username: atracy
    SSH Password: ******************
    MySQL Hostname: 127.0.0.1 (because we're logging in with ssh)
    MySQL Server Port: 3306
    Username: root
    Password: **********

In order to import the database tables we will need to create the scheme present in the previous tables. From our example these would be:

    wordpress
    semi

Right click in the **Schemas** area on the left pane and create the schemas you need. 

Next we will click *Data Import/Restore*

Click the browse button to select the folder where your dump is, and then click start import. This will load the tables into your new database. 

Make sure that you have the same DB users from the old DB on the new DB, this can cause Database Connection Issues if they're not present. Most of the time this user can be found in the **wp-cofig.php**.


### Transfer WP Config Files From Old Host To New Host

Now all that's left to do is transfer our wp config files from the www root to our new host. To achieve this, I will use the rsync command. 

**WWWROOT server1:  /var/www/html**

**WWWROOT server2:  /var/www/html**


Rsync can do an archive sync using the **-a** switch that will keep permissions and file structure and and **-v** switch that will give us some verbose output and tell us what we need to know about the job completion. 

The format of rsync and most unix/linux commands is:

    command [arguments] [source][/source] [destination]

in this case we will assume that we are currently in the */var/www/html/* directory:
{% highlight bash lineos %}
    rsync -av * atracy@server2.domain.com:/var/www/html/
{% endhighlight  %}

Make sure that you have permissions to write to the html directory or your command will fail. You can also run the command from anywhere on server1 with this command:

{% highlight bash lineos %}
    rysnc -av /var/www/html/ atracy@server2.domain.com:/var/www/html/
{% endhighlight  %}
 
Run the ls command and make sure that your permissions and file ownerships stayed the same from one host to the other.

{% highlight bash lineos %}
    ls -lash
{% endhighlight  %}



