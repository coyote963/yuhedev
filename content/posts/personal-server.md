---
title: "Personal Server 1/2"
date: 2021-08-29T20:41:02-04:00
draft: false
---

About a year ago, I purchased a prebuilt refurbished server from [the server store](https://www.theserverstore.com/). The original purpose of this server was to learn more about webhosting and linux. Throughout the 8 months that I've owned this server I've made many mistakes but also learned a lot about best practices. This document serves half as a informative guide on how to set up a server from scratch and half as a continuously updated document for personal use. The primary purpose is for myself in case something terrible goes wrong and my server needs to be restarted.

# Debian First Install

Following is a guide on how to set up my various components, broken into sections

## Preparing Sudo 

I install a debian buster iso from the debian website, then flash a USB with it, plug in the hard drive and run the installer. I partition and format that drive and install nothing except for the ssh server and basic system utilities. The installer is pretty straightforward and I use only the defaults.

After installing, I restart the server, and log in as root. First thing I do is set up sudo. Sudo which stands for **S**uper **U**ser **DO**, allows system account to run with root privileges. For some reason debian installer didn't add it automatically, under the basic system utilities.  


```apt install sudo```

Then I switch to my normal user:

```su <username>```

## Text editor: Vim (Vi Improved)

I prefer vim as a text editor so that is usually the second thing to be installed. Vim is a great configurable text editor that will come in handy when you need to edit configuration files.

```sudo apt-get install vim```

Add hybrid line numbers (show current line number and for the rest of the lines show relative line numbers) also more useful for editting long non-newline-broken lines of text is to [change the default behavior of j and k](https://stackoverflow.com/questions/20975928/moving-the-cursor-through-long-soft-wrapped-lines-in-vim). 

```file: ~/.vimrc
:set number relativenumber
nnoremap <expr> j v:count ? 'j' : 'gj'
nnoremap <expr> k v:count ? 'k' : 'gk'
```

This is good because jumping up and down lines you can simply type the relative jump

## Static IP

It's inconvenient to need to physically be near the server to use it. This is why ssh exists. But my router changes the server ip from time to time, forcing me to check what is the new IP. This fixes that.

```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug enp0s25
iface enp0s25 inet static
	address 192.168.254.123
	gateway 192.168.254.254
``` 

And that will assign the static IP ending in 123 every single time. Restart the server and ssh into it. 

## Configure disk drives.

My disks were already formatted and partitioned prior to this. To do that use the fdisk utility to correctly partition the drive. Then mkfs.ext4 to create ext4 format on them. Won't go into detail here, since this is up to the person. 

My server boots off an SSD and two HDD for media use. I will make it so that these drives are mounted during boot time. 

First I make my mount points:

```
# mkdir /mymedia
# mkdir /backup
```

Mount the drives, you can find the drive names with the `lsblk` command.

```sudo mount /dev/sdXx /mymedia```

Find the HDD UUID:

`ls -al /dev/disk/by-uuid/`

Then edit the /etc/fstab file by adding the following lines

```
UUID=<UUID_YOU_FOUND_IN_THE_PREV_STEP>	/mymedia	ext4	defaults	0	0
UUID=<UUID_OF_SECOND_DRIVE>	/backup	ext4	defaults	0	0
```
 
Reboot and your drives should be there

## Install Self Hosted software

I usually install Transmission, Jellyfin and Docker first. (Since I don't run Jellyfin or Transmission inside a container) There will be a new document detailing how I create a set up script for these on a later date on part 2.

# Closing Thoughts

Hopefully this is useful for other linux users who intend to set up their own server, or at the very basic level a checklist for first things to configure in a system. In part 2, I will document how I run a few full stack web application on it with docker. 
