---
layout: post
title: "shellshock"
category: security
tags: [security, shellshock, bashbug, bashbleed exploit]
---
{% include JB/setup %}

As many of you probably heard today, a new severe bug has surfaced and effects numerous UNIX machines, be it either servers or desktops. Any UNIX machine that uses a bash shell. Experts claim it's actually worse than the heartbleed bug, as it affects much more machines.
The bug goes by the name shellshock, bashbug or bashbleed.

<!--more-->

But anyways, now onto the details, why this exploit is such a big problem.

> A flaw was found in the way Bash evaluated certain specially crafted environment variables. An attacker could use this flaw to override or bypass environment restrictions to execute shell commands. Certain services and applications allow remote unauthenticated attackers to provide environment variables, allowing them to exploit this issue.

This is quite problematic, if an attacker used this bug to execute malicious code.
A lot of Linux distributions, among RedHat who discovered/reported this bug, have already released a patch for this issue.

To find out if you're still affected from this bug, best run this in your bash.

    env HOSTNAME='() { :;}; echo vulnerable' bash -c "env | grep HOSTNAME"

If your shell returns the following, you're still affected and should immediately run an update on your machine.

    vulnerable 
    HOSTNAME=() {  :

<br>
If you've already patched your system, the very same command will return the following:

    env HOSTNAME='() { :;}; echo vulnerable' bash -c "env | grep HOSTNAME"
    bash: warning: HOSTNAME: ignoring function definition attempt
    bash: error importing function definition for 'HOSTNAME'

<br>
Systems that are probably most vulnerable by this bug are servers that are exposed to the internet, e.g webservers, dns servers, mailservers, etc..
Apparently also remote execution of malicious code through cgi-bin is also possible.

my sources/references were:

[Erratasec](http://blog.erratasec.com/2014/09/bash-bug-as-big-as-heartbleed.html#.VCRe0a1QphF)
<br />
[Red Hat Customer Portal](https://rhn.redhat.com/errata/RHSA-2014-1293.html)
<br />
[Red Hat Security Blog](https://securityblog.redhat.com/2014/09/24/bash-specially-crafted-environment-variables-code-injection-attack/)
<br />
[The Verge](http://www.theverge.com/2014/9/24/6840697/worse-than-heartbleed-todays-bash-bug-could-be-breaking-security-for) 
<br />
[Ars Technica](http://arstechnica.com/security/2014/09/bug-in-bash-shell-creates-big-security-hole-on-anything-with-nix-in-it/)
