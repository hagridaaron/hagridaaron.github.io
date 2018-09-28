---
layout: post
title: SUDO in style
---

##SUDO and the meaning of life.

If you've been around linux or mac for long, then you've likey seen the word *sudo* somewhere in the mix. Where does the term come from? What does it mean? My trusty companion [@vicenac](http://twitter.com/vicenac) and I had a little debate about this today.

###SUDO Demystified 

In linux you can use sudo to run a command with root privilidge. This comes in handy if you're security minded and don't execute commands as root very often.

    sudo service restart nginx

or, if you want to become root:

    sudo su -

..This is all good and well, but what's actually going on? Do you pronounce it SUE-DOH or SUE-DO, what are we doing? There's also a group in some distributions of linux called sudoers. What do we make of all this?

SUDO - Super User DO.

Plain and simple. The *"root"* user is also known as super user, so you're telling the shell to run the command as the super user. 

Now, the next time you get into an argument with your favorite Romanian you will have the answer!
