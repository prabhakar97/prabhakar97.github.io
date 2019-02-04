---
layout: post
title: "Boot linux from grub rescue prompt"
date: 2018-01-14 20:20:50 +0530
comments: true
tags: [linux]
---
If you have a dual boot setup of a Linux based OS and Windows 10, and you have setup grub to choose which OS
to boot; you might have experienced the *grub rescue prompt* which comes up after a Windows 10 update screws up
with the boot files.

Panic not, for it's easy to get back to your beloved Linux distro and fix grub. Here are the steps.

* Find out the Linux partition (skip, if you already know)
    - Run `ls` and it should show you a list of partitions like `(hd0,msdos1) (hd0,msdos2) ...` or in the form `(hd0,gpt1), (hd0,gpt2) ...`
    - Run ls with the name of the partition followed by a / to see the files on the partition. Like so `ls (hd0,gpt1)/`. Remember, that forward slash is important. You have to do this for each parition until you find your linux partition - i.e. you see the list of files named `dev, proc, usr, etc, mnt` etc.
* Set the grub modules prefix
    - Run this `set prefix=(hd0,msdos3)/boot/grub`. This assumes, your Linux partition in previous step was msdos3.
* Set the root partition
    - You can set the linux partition as your root partition by running `set root=(hd0,msdos3)`
* Load the needed modules
    - We need to load the linux module to be able to boot Linux. Run `insmod linux` to do that.
* Find where your kernel and initramfs are located
    - You can run `ls /boot/` and it should show a file named `vmlinuz-linux`. That's your Kernel. At least that's how it is named in Arch Linux and related distros like Antergos and Manjaro. Kernel and Ramdisk have different names on different distros. Centos, Fedora and RHEL have names similar to `vmlinuz-3.10.0-693.11.1.el7.x86_64` for the Kernel. I assume you are smart enough to figure the naming out.
    - You can similarly find your initramfs image. Usually it is named something like `initramfs-linux.img`.
    - Caveats to look out for when finding the Kernel and initramfs image files - Number one, if the file names are versioned, choose the latest ones. Number two, choose the same version for Kernel and initramfs. And the last and number three, don't choose the names that contain `rescue` or `memtest`. They are not the ones we are interested in.
* Boot Linux!
    - Having found out the Kernel and Ramdisk images, let's load them up. Run `linux /boot/vmlinuz-linux root=/dev/sda3 fb rw quiet`. sda3 can probably be hda3 if you have an older machine. And the digit at the end should be same as the root partition number we found in the first step i.e. msdos3 in my case. Now load the Ramdisk by running `initrd initramfs-linux.img`.
    - Having loaded 'em both, let's boot. Run `boot` and bingo, your system should now boot normally!
* Fix grub so that you don't have to go through this ordeal next time
    - Running `sudo grub-install /dev/sda` or in some cases hda instead of sda, should reinstall grub correctly and fix your booting problems.



