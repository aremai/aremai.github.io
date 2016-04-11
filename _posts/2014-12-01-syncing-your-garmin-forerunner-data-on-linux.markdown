---
layout: post
title: "Syncing your Garmin Forerunner data on linux"
tags: [garmin, gps, gps watch, gpx, forerunner]
category: misc
excerpt_separator: <!--more-->
---

hi there,

it's been a while.

i have finally upgraded my linux desktop to arch linux, and wanted to sync my latest Garmin Forerunner data with my linux.
it's quite simple if you follow these next steps.

<!--more-->

first you'll need a bunch a packages installed.

```ruby
    sudo pacman -S python2
    sudo pacman -S python2-pip
    sudo pip2 install --pre pyusb
    sudo pip2 install pyserial
    sudo pip2 install lxml
```

Afterwards you'll only need to do:

```ruby
    sudo pip2 install distribute
    sudo pip2 install python_ant_downloader
```

Now if you plug in your Garmin ANT+ USB stick and run "lsusb" you should see something like that:

```ruby
    Bus 003 Device 006: ID 0fcf:1008 Dynastream Innovations, Inc. ANTUSB2 Stick
```

We need this information to create a udev rule, in order to mount non-standard usb sticks automatically. Those devices follow permission rules that are present in /etc/udev/rules.d/.

The following command creates such a udev rule for our ANT+ usb stick.

```ruby
    sudo bash -c "echo 'SUBSYSTEMS==\"usb\", ATTRS{idVendor}==\"0fcf\", ATTRS{idProduct}==\"1008\", MODE=\"666\"' > /etc/udev/rules.d/99-garmin.rules"
```

After this, you're good to go. You have your USB stick connected to the laptop/pc and then can run the following command

```ruby
    ant-downloader
```
On the inital run it can take a while till it actually grabs your data, because it generates a config called ~/.antd/antd.cfg
In this directory you'll also find a sub-directory called 0xDEVICENR/tcx/  where all your workouts are stored in raw and tcx format.

As a final step I usually upload my workouts (tcx) to a website called [Runningahead.com](http://www.runningahead.com) but you might as well upload it to your garmin account directly.

Hope you like it as much as I do. ;)
