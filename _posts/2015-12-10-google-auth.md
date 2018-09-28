---
layout: post
title: Adding Google Authenticator to Ubuntu 14.04
---
###In this article we will discuss and walk through an installation of the Google Authentication PAM module on Ubuntu Linux 14.04

Make sure your apt-get is up to date

    sudo apt-get update

install the libpam google authenticator package
   
    sudo apt-get install libpam-google-authenticator

login with the user you would like to force the authentication module and run the google-authenticator command.

    TestMachine@TestUser:~$ google-authenticator

Follow the steps listed and make sure to be thoughtful when answering questions like login attempts, rate-limiting and such.

You will be given a QR code to scan with your phone (with the Authenticator app installed) this will setup OTPs for your two factor authentication.

We will now require Google Authenticator for SSH Logins:

open **/etc/pam.d/sshd** and add the following line to the file:

    auth required pam_google_authenticator.so

Next, open the /etc/ssh/sshd_config file, locate the ChallengeResponseAuthentication line, and change it to read as follows:

    ChallengeResponseAuthentication yes

(If the ChallengeResponseAuthentication line doesn't already exist, add the above line to the file. Finally, restart the SSH server so your changes will take effect:

    sudo service ssh restart
