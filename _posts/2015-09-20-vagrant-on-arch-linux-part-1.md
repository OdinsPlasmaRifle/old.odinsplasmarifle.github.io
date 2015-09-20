---
layout: post
title: Vagrant on Arch Linux - Part 1
comments: true
---

*A brief guide on how to install and use Vagrant on Arch Linux.*

![Vagrant Banner](/public/images/posts/vagrant_banner.png)

>Vagrant is a tool for building complete development environments. With an easy-to-use workflow and focus on automation, Vagrant lowers development environment setup time, increases development/production parity, and makes the "works on my machine" excuse a relic of the past.

In short, Vagrant is a tool you can use to configure and setup virtual environments. Do you need to test or develop a PHP web application within a Ubuntu 14 environment that emulates its production server? Vagrant will allow you to do this with ease.

In this three part guide I will be providing basic instructions on how to get Vagrant up and running within Arch Linux. In order to illustrate a use-case I will also show how an Ubuntu LAMP stack can be configured and later used within development.

##Install Vagrant

To get started, open a teminal and run the following:

    $ sudo pacman -S vagrant

This will install the Vagrant software via pacman, the Arch package manager.

##Install VirtualBox 

Seeing as Vagrant is a wrapper for virtualization software, rather than the software itself, we will also need to install additional packages.

For this example we will use VirtualBox. However, if you wish, virtualization can be done through other software such as VMWare, KVM or Linux Containers.

Enter the following command to install VirtualBox:

    $ sudo pacman -S virtualbox

Once the VirtualBox installation is complete, several optional dependencies will be listed. Install them via pacman as well.

##Probe the Kernel Modules

Next, run these commands to install and probe the kernel modules

    $ sudo dkms autoinstall
    $ sudo modprobe vboxdrv
    $ sudo modprobe vboxnetadp
    $ sudo modprobe vboxnetflt
    $ sudo modprobe vboxpci

Because you want the above to run on start up you will have to create a configuraion file that triggers on load. So, using your text editor of choice (vi, vim, nano etc.) create and edit a file at this location:

    $ vim /etc/modules-load.d/virtio-net.conf

Add the following lines to the file you just created:

    vboxdrv
    vboxnetadp
    vboxnetflt
    vboxpci

Save and exit the file. This new file will simply probe the modules every time Arch starts up.

This finishes teh first step of installing Vagrant. You should now have a fully functional Vagrant installation with all the necessary VirtualBox files and kernel modules.

* * *

See **Part 2** of this guide to learn how to set up an Ubuntu virtual box using Vagrant.

###Further reading:

[Vagrant docs](https://docs.vagrantup.com/v2/)

[Arch Vagrant docs](https://wiki.archlinux.org/index.php/Vagrant)

[Arch VirtualBox docs](https://wiki.archlinux.org/index.php/VirtualBox)
