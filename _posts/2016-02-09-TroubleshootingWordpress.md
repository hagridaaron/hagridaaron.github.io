---
layout: post
title: Troubleshooting Misc WordPress Issues 
---

# Things not working on your WordPress instance that really should be? You've come to the right place. 

There are many things that can go ah-rye when configuring WordPress without a careful eye for detail. Permalinks, Permissions, Updates, and FTP access can all go bad if you don't watch out. This is a reference you can use after things have gone south. 

## Permalinks Not Working
A few things that could be going wrong:

* permissions
* .htaccess
* apache configuration

### Permissions
Permissions play a large part in the life of your WordPress installation. It's important to note that you can go as far down the permissions rabbit hole that you want to go. If you've got an itch for that, hopefully [this article](https://codex.wordpress.org/Changing_File_Permissions) can help you scratch it. Suffice it to say that you should understand how Unix/Linux permissions work, and what implications they have for the security and usability of your WP instance. 

The owner of your instance, hopefully nobody:nogroup, as well as your webserver user (possibly www-data, apache, or httpd) will need to be able to write to the root of your WP instance, and likely other sub-directories. For instance:

    nonrootuser@webserver1:/var/www$ ls -lash
    4.0K drwxr-xr-x  5 nobody nogroup 4.0K Feb  9 14:32 wordpress

The user nobody can write, and everyone else can read and execute. It's important to note that these might not be the most secure permissions, but this is what i'm using for examples sake.


### .htaccess
WordPress *SHOULD* automatically make this file for you, if you have gone over the above permissions, or yell at you if it cant. Though, it can never hurt to cover all your bases. Your /var/www/wordpress/.htaccess should look something like this:

    <IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteBase /
    RewriteRule ^index\.php$ - [L]
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule . /index.php [L]
    </IfModule>

*Note* if you haven't configured your web-server (apache for this article) to use Mod_rewrite, then you're going to have a bad time. Keep reading.


### Apache Configuration - Enabling mod_rewrite 
If you're on a Debian based system like Ubuntu, you will likely be using the package apache2 instead of the httpd service on RHEL or CentOS. Inside your httpd.conf or apache2.conf files in /etc/httpd and /etc/apache2 respectively you will need to make sure that the rewrite module is turned on. 

##### Apache2
We can use the following apache2 command to enable mod_rewrite:

    a2enmod rewrite


##### httpd
Make sure that the following line is uncommented in your */etc/httpd/httpd.conf* file:

    LoadModule rewrite_module modules/mod_rewrite.so

After we've made sure Mod_Rewrite is enabled, you can add something similar to the snippet blow to your .conf file:

    <Directory /var/www/wordpress/>
    Options Indexes Includes FollowSymLinks Multiviews
    AllowOverride All
    Order allow,deny
    Allow from all
    </Directory>




## Updates/FTP issues
Along with permalinks, FTP and auto-update problems are ubiquitous in the WP world, things that can mess you up are:

* Local FTP User
* FS_Method Definition

### FTP User
It's important to note that your WordPress installation might be in need of a ftpuser or something like an ftp user. This local server account is what you can give permissions to write in specific directories within your site. The ftpuser or similar can be defined in your wp-config.php: Make sure you use your **;**!

    define('FTP_USER', 'ftpuser');
    define('FTP_PASS', 'password');
    define('FTP_HOST', 'localhost');

This definition will show WordPress how to write to your *"ftp_host"*

### FS_Method and wp-config.php
Another problem that can be involved with FTP problems is automatic updates or one-click updates from within the WordPress GUI.

Make sure that you have the correct FS_Method definied in your wp-config.php. If you want automatic one-click updates to be turned on, give the direct method a shot:

    define ('FS_METHOD','direct');


