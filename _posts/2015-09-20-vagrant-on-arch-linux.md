---
layout: post
title: Vagrant on Arch Linux
comments: true
---

*A brief guide on how to install and use Vagrant on Arch Linux.*

![Vagrant Banner](/public/images/posts/vagrant_banner.png)

>Vagrant is a tool for building complete development environments. With an easy-to-use workflow and focus on automation, Vagrant lowers development environment setup time, increases development/production parity, and makes the "works on my machine" excuse a relic of the past.

##Install Vagrant

To get started, open a teminal and run the following:

    $ sudo pacman -S vagrant

This will install the Vagrant software via pacman, the Arch package manager.

##Install VirtualBox 

Seeing as Vagrant is a wrapper for virtualization software, rather than the software itself, we will also need to install some sort of virtualization software.

For this example we will use VirtualBox. However, virtualization can be done through other software such as VMWare, KVM or Linux Containers.

Enter the following command to install VirtualBox:

    $ sudo pacman -S virtualbox


Once the VirtualBox installation is complete, a couple optional dependencies will be listed. Install them via pacman as well.

##Probe the Kernel Modules

Next, run these commands to install and probe the kernel modules

    $ sudo dkms autoinstall
    $ sudo modprobe vboxdrv
    $ sudo modprobe vboxnetadp
    $ sudo modprobe vboxnetflt
    $ sudo modprobe vboxpci

Because you want the above to run on start up you will have to create a configuraion file that triggers on load. Using your text editor of choice (vi, vim, nano etc.) create and edit a file at this location:

    $ vim /etc/modules-load.d/virtio-net.conf

Add the following lines inside virtion-net.conf:

    vboxdrv
    vboxnetadp
    vboxnetflt
    vboxpci

Save and exit the file. This new file will probe the modules every time Arch starts up and loads its modules.

##Setup the Vagrant Box

At this point we have completed the basic setup of Vagrant and we can move to usage. As an example I will show you how to setup a simple Ubuntu server with a LAMP stack pre-installed.

First, either build or download the Vagrant config files for the VM setup. There are plenty versions of an Ubuntu Trusty server but in this exmaple we will use the following repository:

https://github.com/mattandersen/vagrant-lamp

Download the above files.

Now, in your home directory (eg. /home/{your_name}/) make a folder called 'Webserver'.

Navigate into webserver and add 2 folders:  'trusty' and 'sites'. 

Move or copy the following files from the vagrant-lamp download folder to the trusty folder you just created:

    Vagrantfile
    provision.sh

Now, in order to make the vagrant box useful for development we need to make a few configuration changes, You can do this by editting the above 'Vagrantfile' to look like:

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


The above configuration changes allow for a synced path between the Virtual Box and the local computer hosting the box. The synced paths are on this line:

    config.vm.synced_folder "../sites", "/var/www/html/"]

The other addition to the file enables networking and gives the Virtual Box an IP:

    config.vm.network "private_network", ip: "192.168.56.55"

##Install the Virtual Box

You will now have to install the Vagrant box. This can be done via single command. Navigate into the 'trusty' folder we created earlier and run this:

    $ vagrant up

This command will install the box with Ubuntu and any other requirements specified in the provisions file (Apache, PHP, etc.).

Once the command is complete it will also run the vritual box so that you can imemdiately start making use of it. In future you will use the same '$ vagrant up' command to just run the already installed box.

##Vagrant Usage

The VirtualBox has now been fully installed and we can now move onto its usage.

To run a virtual box, use the follwong command (inside the trusty folder):

    $ vagrant up

To SSH into the Vagrant box use: 

    $ vagrant ssh

If you would like to shutdown the Virtual Box, run this:

    $ vagrant halt

The above command will attempt a graceful shutdown of the Virtual Box.

If you want any files to be synced between your local and the vm place them in:

/home/{your_name}/webserver/sites/

As per the 'Vagrantfile', any work done in '/home/{your_name}/webserver/sites' will automatically replicate in the Virtual Box here:

/var/www/html/

##LAMP usage

Now, seeing as the goal of the virtual box is to allow for a 'Real Life' server environment while still developing locally we will want a way to access the VM in our local browser. To do this we need to first set up a project's files in the '/home/{your_name}/webserver/sites' directory. 

Next, in Virtual Box itself, you will want to set up a virtual host for the above project. This can be done by SSHing onto 'trusty' and creating a virtual host via Apache. To SSH onto 'trusty' navigate to its folder and enter:

    $ vagrant ssh

If you need help with how to set up a virtual host, see the help links included at the end of the Article. Otherwise, create a virtual host for yoru project.

Next, you will have to edit your local hosts file (not in the virtual box). Set it to something like:

    192.168.56.55 {your_website_url}

Notice how we are using the IP we set in the VagrantFile earlier.

Finally, open your browser of choice, and navigate to {your_website_url}. If the above steps were followed correctly you should now see your website as hosted on the Virtual Box.

###Further reading:

[Vagrant docs](https://docs.vagrantup.com/v2/)

[Apache Virtual Hosts](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts)

[Arch Vagrant docs](https://wiki.archlinux.org/index.php/Vagrant)

[Arch VirtualBox docs](https://wiki.archlinux.org/index.php/VirtualBox)
