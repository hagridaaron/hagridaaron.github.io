---
layout: post
title: SSH Key Management 
---
If you've done any Mac or Linux administration you know that using SSH is handy, but can sometimes be a pain if you're using loads of passwords (which you should be). SSH Keys help us with that problem. Using an SSH Keypair you can securly login to any ssh-enabled machine without a password if you configure it correctly. Make sure to use this with discretion, as passwords and authentication are there for a reason. The process for setting this up is very basic: you have a list of servers **(~/serverlist.txt) and the below script. The commands will import your public SSH key into the server's *authorized_keys* file, and automatically authenticate you when running ssh.

*note: this script makes use of a utility called ssh-copy-id. If you're on an older system you might need to google around for a different solution involving the authorized_hosts file in the users home directory. 

{% highlight bash lineos  %}
    #!/bin/bash -x
    # Imports SSL Key to remote servers
    # Author Aaron Tracy
    for host in $(cat ~/serverlist.txt); do
     ssh-copy-id -i ~/.ssh/id_rsa.pub atracy@$host
    done
    exit 0
{% endhighlight  %}

After you've imported all your ssh keys can can use ssh normaly:

    ssh atracy@server.domain

If you manage multiple servers, find youself doing repetitive commands or writing bash scrips to manage servers you should check out the free automation tool [Ansible](http://ansible.com). I'll get into Ansible in more depth in a later post. 

hagridaaron
