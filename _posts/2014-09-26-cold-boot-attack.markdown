---
layout: post
title: "Cold boot attack"
tags: [cold boot attack, hack, full-disk-encryption]
category: encryption
excerpt_separator: <!--more-->
---
{% include JB/setup %}

Ever thought that full-disk (hardware) encryption would provide full safety to your precious data? Well, then you thought wrong ;-)

<!--more-->

## the problem

The cold boot attack is a side channel attack in which an attacker, with physical access to the target computer, reads contents of memory modules after the system was turned off.

It is based on the data remanence in common RAM modules, in which electric charge under certain conditions (or solely due to manufacturing tolerances) slowly dissipates the data contents of the memory cells can still be successfully read out completely (even after a couple of minutes). Cooling the memory modules increases the remanence drastically. Treating the modules with cooling spray can help keep the contents available for several minutes.

In order for the attack to work the targeted computer is booted cold into a minimal operating system, since this mini-system consumes very little memory. Thus it will leave much of the memory untouched, ensuring it still contains the same content that was there before it got restarted.

You can then extract the cryptographic keys to encrypt data that was accessed just before the crash occured. This could be, for example, the key of full-disk encryption systems.

## countermeasures

A best practice to mitigate the attack opportunities is to overwrite the key when you remove the disk (for example, when halting the system), so that the data at least is safe. As a countermeasure the Trusted Computing Group recommends in the "TCG Platform Reset Attack Mitigation Specification" that the BIOS purges the content of the memory at power-on self-test, when an unclean termination of the operating system has been detected.

One way to fix the underlying problem is to hold the key only in the processor's registers. For Linux based-systems (x86_64) and Android (ARM) implementations of this approach is part of the kernel and is available in the form of a patch.
