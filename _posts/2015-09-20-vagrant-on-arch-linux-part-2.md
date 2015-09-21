---
layout: post
title: Vagrant on Arch Linux - Part 2
comments: true
---

*Continuation of the 'Vagrant on Arch Linux' guide.*

![Vagrant Banner](/public/images/posts/vagrant_banner.png)

In [Part 1]({% post_url 2015-09-20-vagrant-on-arch-linux-part-1 %}) I gave instructions on how to install Vagrant and VirtualBox on Arch Linux. In [Part 2]({% post_url 2015-09-20-vagrant-on-arch-linux-part-2 %}) I'll give step by step instructions on how to set up a full Ubuntu (Ubuntu 14.04 Trusty Tahr) Vagrant box.

##Set up the Vagrant Box

The first step is to either build or download the Vagrant config files for the Vagrant setup. For simplicities sake we'll be creating an Ubuntu Virtual Box that includes a LAMP stack. There are plenty pre-compiled vagrant files for Ubuntu 14 but for this exmaple we will use the following repository:

    https://github.com/mattandersen/vagrant-lamp

Download the above files and unzip them wherever you prefer.

Now, in your home directory (eg. /home/{your_name}/) make a folder called 'Webserver'.

Navigate into webserver and add 2 folders:  'trusty' and 'sites'. The 'trusty' folder will house the vagrant config files. While the 'sites' folder will be a location that is synced between localhost and the Virtual Box.

Next, move or copy the following files from the vagrant-lamp folder you downloaded above to the 'trusty' folder you just created:

    Vagrantfile
    provision.sh

In order to make the Vagrant Box useful for development we need to make a few configuration changes, You can do this by editting the 'Vagrantfile' to look like this:

    # -*- mode: ruby -*-
    # vi: set ft=ruby :

    # Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
    VAGRANTFILE_API_VERSION = "2"

    Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
        config.vm.box = "ubuntu/trusty64"

        # Forward ports to Apache and MySQL
        # config.vm.network "forwarded_port", guest: 80, host: 8888
        # config.vm.network "forwarded_port", guest: 3306, host: 8889
        config.vm.network "private_network", ip: "192.168.56.55"

        config.vm.synced_folder "../sites", "/var/www/html/"

        config.vm.provision "shell", path: "provision.sh"

      config.vm.provider "virtualbox" do |vb|
      #   # Don't boot with headless mode
      #   vb.gui = true
      #
      #   # Use VBoxManage to customize the VM. For example to change memory:
          vb.customize ["modifyvm", :id, "--memory", "2048"]
      end
    end

The above configuration updates allow for a synced path between the Virtual Box and the local computer hosting the box. The paths that will be synced are located on this line:

    config.vm.synced_folder "../sites", "/var/www/html/"

The other addition to the file enables networking and gives the Virtual Box an IP:

    config.vm.network "private_network", ip: "192.168.56.55"

##Install the Virtual Box

You will now have to create the Virtual Box using Vagrant. This can be done via a single command. Navigate to the 'trusty' directory and run this:

    $ vagrant up

This command will create an Ubuntu box and install any other requirements specified in the provisions file (Apache, PHP, etc.). Now might be a good opportunity to take a moment to look at the 'provisions.sh' file so you can get a better understanding of how Vagrant actually does its thing.

Once the command is complete it will also run the virtual box so that you can imemdiately start making use of it. In future you will use the same '$ vagrant up' command to just run the already installed box.

* * *

See [Part 3]({% post_url 2015-09-20-vagrant-on-arch-linux-part-3 %}) of this guide to learn how to make use of Vagrant on your new virtual box.

###Further reading:

[Vagrant docs](https://docs.vagrantup.com/v2/)

[Arch Vagrant docs](https://wiki.archlinux.org/index.php/Vagrant)

[Arch VirtualBox docs](https://wiki.archlinux.org/index.php/VirtualBox)