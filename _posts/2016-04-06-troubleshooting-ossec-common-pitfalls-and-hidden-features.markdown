---
layout: post
title: "Troubleshooting OSSEC -- common pitfalls and hidden features"
tags: [OSSEC, HIDS, UNIX]
category: ossec
excerpt_separator: <!--more-->
---

Hi *,

it's been a while since I last blogged, mainly busy with work and university. Good news though, I have finished my first thesis on OSSEC and PCI Compliance, and will probably upload it here at some point.

I just came up with the idea about writing a blog post about common pitfalls that I and almost every other OSSEC user will run into at some points.

<!--more-->

So, consider this blog post to be a list of tips to troubleshoot OSSEC and how to solve it.

At this point I should mention that some of those problems may seem trivial to you, but maybe other find it useful and saves them some time googling....

### 1.  __tail: cannot watch `/var/ossec/logs/ossec.log': No space left on device__

ok, this is an easy one. However, the error message is a bit irritating because it indicates that the filesystem is full, which it clearly is not. Instead your system (or better said: the kernel) has reached the inotify watch limit.

You can run a 

     ``` # inotifywatch -v /var/ossec/logs/ossec.log
           Failed to watch /var/ossec/log/ossec.log; upper limit on inotify watches reached!
           Please increase the amount of inotify watches allowed per user via '/proc/sys/fs/inotify/max_user_watches'.` ```

I ran this command to verify it:

     ``` # cat /proc/sys/fs/inotify/max_user_watches
           8192 ```


and then ran this command to increase it permanently

     ``` # cat /proc/sys/fs/inotify/max_user_watches
           524288 ```


To find out what's using up your inotify watches, run this command:

      ``` # for foo in /proc/*/fd/*; do readlink -f $foo; done |grep inotify |cut -d/ -f3 |xargs -I '{}' -- ps --no-headers -o '%p %U %c' -p '{}' |uniq -c |sort -nr

            2     1 root     init
            1   399 root     udevd
            1 21291 root     ossec-syscheckd
            1  1581 root     udevd
            1  1580 root     udevd
            1  1475 root     crond 
	    ```

So you can see, the syscheckd process is using a lot of inotify watches for realtime-alerting (real time file integrity monitoring).


I will edit this blog post and continuously add new tips to troubleshoot OSSEC.
Shoot me an email with your OSSEC troubleshooting tips, or problems you ran into ... looking forward to reading them!
