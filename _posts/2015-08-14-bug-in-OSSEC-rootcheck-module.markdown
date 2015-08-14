---
layout: post
title: "Bug in OSSEC module"
tags: [ossec, security, ids, hids, centOS]
category: ossec
---

hi,

i've been playing around with OSSEC's rootcheck module, which enables you to run system audits and system policy enforcement in the likes of CIS benchmark tests.
in my previous blog post i briefly mentioned the CIS benchmark tests, which is basically just a framework of system checks. it scans your system for weaknesses that could be potentially exploited by intruders. the main focus is all things system security related, goes from SELinux configuration to secure boot settings and legacy system services (e.g telnet, rlogin, tftp and the likes). i think it helps a sysadmin to harden their systems by pointing out possible access points into the system -- it certainly is a good guideline.

however back to the topic. as mentioned i've been running a few rootchecks and noticed a weird behavior. at that point i must say i haven't been running the most recent version that is available on github instead i was running the latest stable version that is available through atomicorp's yum repository. the version was 2.8.2

<!--more-->

the problem i faced were odd segmentation faults caused by this command:

    # /var/ossec/bin/rootcheck_control -L -i 000

the -L switch lists the time of the last check and its results, and with the -i switch you can specify the host (000 = the ossec master, 002 would be an agent)
as soon as i ran the check it threw a segmentation fault. this was starting to worry me. what could possbily cause a segmentation fault?

i checked the system logs as well, and found the following error:

    # Aug 10 18:45:23 tron kernel: rootcheck_contr[20641]: segfault at 8 ip
      00007f851fe8b925 sp 00007ffdc8c73240 error 4 in
      libc-2.12.so[7f851fde8000+18a000]

i then tried to debug it with strace to see which system call is problematic.
strace quickly identified the problem...

    # open("/etc/localtime", O_RDONLY)        = 4
      fstat(4, {st_mode=S_IFREG|0644, st_size=2211, ...}) = 0
      fstat(4, {st_mode=S_IFREG|0644, st_size=2211, ...}) = 0
      mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7ffb97d01000
      read(4, "TZif2\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\6\0\0\0\6\0\0\0\0"..., 4096) = 2211
      lseek(4, -1410, SEEK_CUR)               = 801
      read(4, "TZif2\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\7\0\0\0\7\0\0\0\0"..., 4096) = 1410
      close(4)                                = 0
      munmap(0x7ffb97d01000, 4096)            = 0
      --- SIGSEGV {si_signo=SIGSEGV, si_code=SEGV_MAPERR, si_addr=0x8} ---
      +++ killed by SIGSEGV +++

apparently the localtime couldn't be identified through /etc/localtime, which usually checks when the last rootcheck had been performed.
i then contacted one of the main ossec-core developer to help me fix this problem. we did a few tests to verify and also to narrow it down. according to him the problem didn't occur on debian based systems.
so the bug must either be system (centos) related or a bug somewhere in the ossec code. we assumed the first one, without comparing the ossec-versions, as it was working on his test environment.
after some checks we quickly located that the problem must be in the way localtime is called in ossec's code, namely in the read-agents.c file.
thankfully this bug was already addressed and fixed in the latest version that's available on github. therefore i uninstalled the RPMs and installed the latest version.

so far i haven't had any problems, no more segfaults when running the rootcheck.

for anyone still running ossec's 2.8.2 version and centos 6.x you might want to upgrade your ossec installation.

i also informed the folks at atomicorp about the bug....hopefully they will soon update their rpm's accordingly.


   best,
   theresa
