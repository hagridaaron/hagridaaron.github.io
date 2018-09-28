---
layout: post
title: Reverse Proxy Server using NGINX
---
##Reverse Proxy User Guide
####RP = Reverse Proxy
The current RevProxy1.fuller.edu is built on Centos7 and uses Nginx as it's proxy server. _The setup for this internal RP is very similar to our cloud RP located at 66.110.189.96._ Simply put the reverse proxy works by consuming http requests bound for internal servers and then proxies the communication between client and server. We do this by pointing server hostnames in DNS to the reverse proxy's IP address. 

##DNS

Lets use **webserver.fuller.edu** as an example. in the DNS the entry looks like this:
  
    66.22.x.x   webserver.fuller.edu@INBOUNDNAT              CNAME=thewebserver.fuller.edu        HINFO="Webserver Production Server"

We will change the ip address *65.118.138.95* to the ip of our RP server.


    192.168.x.x   webserver.fuller.edu@INBOUNDNAT              CNAME=thewebserver.fuller.edu        HINFO="webserver Production Server"

Now all http requests for webserver.fuller.edu on the internal fuller network will point to **192.168.x.x**

## Nginx Config
Now we have requests coming into our RP server, now we need to get them out to the correct servers and proxy the traffic between client and server. To do this we have some nginx config files. specifically:

    /etc/nginx/nginx.conf
    /etc/nginx/conf.d/

Inside the nginx.conf file we will add a line called include, which will allow us to place configuration files in the directory /etc/nginx/conf.d/ and have them added to the main nginx.conf that the server reads. 

{% highlight nginx lineos  %}
    include /etc/nginx/conf.d/*.conf;
{% endhighlight  %}

After we include the conf.d directory we can start to build out our RP config files. I like to make server specific files for each new server that will be proxied so changes can be made directly to that file instead of digging through one huge configuration file. for example:

    /etc/nginx/conf.d/webserver.http.conf
    /etc/nginx/conf.d/404.http.edu
    /etc/nginx/conf.d/redisapp.http.conf

It is not necessary to put the .httpd on the file name, but it can become helpful if you start making httpd and https conf files. 

###RP Specific Configuration Files
Below is an example of the config file used for **webserver-dev** on the internal RP server.  

{% highlight nginx lineos %}  
    server{
       listen 80;
       server_name webserver-dev.fuller.edu;
       access_log /var/log/nginx/proxy_access.log;
       error_log /var/log/nginx/proxy_error.log;
       location / {
          proxy_pass http://192.168.72.96/;
          proxy_redirect off;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_max_temp_file_size 0;
          client_max_body_size 10m;
          client_body_buffer_size 128k;
          proxy_connect_timeout 90;
          proxy_send_timeout 90;
          proxy_read_timeout 90;
          proxy_buffer_size 4k;
          proxy_buffers 4 32k;
          proxy_busy_buffers_size 64k;
          proxy_temp_file_write_size 64k;
       }
    }
{% endhighlight %}

##Add New Host To RP Server
On the host create a file called servername.http.conf in the **/etc/nginx/conf.d/** directory

    vim /etc/nginx/conf.d/servername.httpd.conf

you can now paste the above config into this new file. Make sure to change the *server_name* and *proxy_pass* directives if you're going to copypasta. 

After creating the new configuration you'll need to restart the httpd service. There are a few ways to do this, neither one is better than the other, although some may work on some systems, others may not. 

####Centos 7 (Systemctl)
{% highlight bash lineos  %}
    systemctl restart nginx
{% endhighlight  %}
####Centos 7 (Service)

{% highlight bash lineos  %}    
    service nginx restart
{% endhighlight  %}

####Centos <=6 
{% highlight bash lineos  %}
    /etc/init.d/nginx restart
{% endhighlight  %}

####Ubuntu/Debian Systems
{% highlight bash lineos  %}

    service nginx restart
{% endhighlight  %}

## Troubleshooting

After restart you should be good to go. If you get errors, you can use the following command to take a look at the nginx error log file (this presupposes that your logs are going to /var/log/nginx/)

{% highlight bash lineos  %}
    tail -f /var/log/nginx/error.log
{% endhighlight  %}

This will give you a live view of the log file, it might be useful to use the less command if you're getting heaps of errors. This will allow you to scroll up and down and search with the / character.
{% highlight bash lineos  %}
    less /var/log/nginx/error.log
{% endhighlight  %}

Most of the time you will have syntax issues, or maybe permissions issues. After checking/fixing the issue go back and restart the service.

