---
layout: post
title: "OSSEC (WUI) and SELinux"
tags: [ossec, security, ids, hids, centOS]
category: ossec
---

hi,

it certainly has been a while since my last post but for a very good reason. recently i've been playing around with OSSEC, a very cool host-ids.
it took a while until i had a working lab environment, but now i'm all set up (still learning) but also making a lot of progress as i go and thus would love to share my issues and troubles i've run into.
maybe this can be of help to someone who's currently in the same pitfall.

however, i should stop rambling and get to it.

today i want to share my experience regarding ossec and selinux, and how they work together (or in my case don't work together).

my test environment is a standard **centos (6.7) box**, having installed the latest patches et al, and it comes with selinux enabled by default. now what most people do is to disable selinux right away. well, i can understand this to some extent, because selinux can be such a hassle if you just want a working environment and/or application. BUT for security sanity we would want selinux ENABLED for a very good reason. i know it can be a pain in the neck sometimes, but what OSSEC can also do is, check your system policies (OSSEC ships with a rootkit and audit check module for policy enforcement). one of those checks is the so-called CIS check (google CIS benchmark if you don't know what it does).
one of the CIS benchmark tests checks if you have selinux enabled and if you have it set to "targeted" mode. since i wanted my system to pass those two CIS benchmark checks i fiddled around with selinux a bit to get it working.
hopefully i could now convince you to enable selinux on your box again.

here's what i did to get ossec and its deprecated web-ui (WUI) working with selinux enabled.

<!--more-->

first of all i cloned the latest version of ossec-wui from the [github repository](https://github.com/ossec/ossec-wui), and followed its guidelines. i can't stress enough that you follow these guidelines (and adapt the apache groupname "www-data" with "apache" if you're runnig a centOS/RHEL based system) and **ESPECIALLY** use the setup script.



    # cd /var/www/html/ossec-wui
    # ./setup.sh

i forgot to run that, and at first ossec-wui never quite really worked. (should have read the guidelines more carefully).
however after the setup script has finished, you're pretty much good to go. you can call it in your browser, and most likely will be facing a typical HTTP 403 (forbidden) error.
at first i was quite confused, and would have never thought of selinux to be causing the problem.
but a quick look into the apache error logs, i found a lot of "permission denieds" in this log....

    # Warning: opendir(/var/ossec/etc/ossec.conf) [function.opendir]: failed to open dir: Permission denied in /var/www/ossec-wui/lib/os_lib_handle.php on line 94

in order to fix this problem i did the following:

   this restores the security context of the ossec wui folder

    # restorecon -R /var/www/html/ossec-wui/

   and this changes the security context for the OSSEC alerts.log file so it can be read by the apache server. 

    # chcon -R -t httpd_sys_content_t '/var/www/html/ossec-wui/'

    # chcon -t httpd_sys_content_t '/var/ossec'
    # chcon -R -t httpd_sys_content_t '/var/ossec/logs/'
    # chcon -R -t httpd_sys_content_t '/var/ossec/queue/agent-info/'
    # chcon -R -t httpd_sys_content_t '/var/ossec/queue/syscheck'
    # chcon -R -t httpd_sys_content_t '/var/ossec/stats/'

   after that you only need to restart the apache daemon and ossec.

    # service httpd restart
    # /var/ossec/bin/ossec-control restart

   now you should be able to see all your alerts and integrity checks in your ossec-wui instead of the permission denieds or the "unable to access ossec directory" messages.

   if you do run into a problem with selinux enabled, shoot me an email. maybe i can help?!

   best,
   theresa
