---
layout: post
title: Vagrant on Arch Linux - Part 3
comments: true
---

*Continuation of the 'Vagrant on Arch Linux' guide.*

![Vagrant Banner](/public/images/posts/vagrant_banner.png)

In [Part 1]({% post_url 2015-09-20-vagrant-on-arch-linux-part-1 %}) and [Part 2]({% post_url 2015-09-20-vagrant-on-arch-linux-part-2 %}) I addressed installation and configuration of a simple Vagrant box. In this last part of the guide I'll take a look at basic usage as well as one of the many ways you could make use of your Virtual Box for development.

##Vagrant Usage

If you followed the instructions in Part 1 to 2, the Ubuntu Virtual Box should be completely installed. So, now would be a good time to look at some of the basic Vagrant commands. For any adidtional commands, not mentoned here, take a look at the Vagrant docs linked at the end of this article.

All Vagrant commands should be run from within the 'trusty' folder we created in [Part 2]({% post_url 2015-09-20-vagrant-on-arch-linux-part-2 %}).

For starters, to run a virtual box, use the following command:

    $ vagrant up

To SSH into the Vagrant box use: 

    $ vagrant ssh

Bear in mind, for any functionality that may require a password the vagrant username and password are:

    Username: vagrant
    Password vagrant

These details may be used in situations such as when you want to access a database (in the virtual box) via a SSH tunnel.

If you would like to shutdown the Virtual Box, run this:

    $ vagrant halt

The above command will attempt a graceful shutdown of the Virtual Box.

If you want any files to be synced between your local and the virtual box place them in:

    /home/{your_name}/webserver/sites/

As per the 'Vagrantfile', any work done in '/home/{your_name}/webserver/sites' will automatically replicate in the Virtual Box here:

    /var/www/html/

##LAMP usage

Now, seeing as the goal of the virtual box is to allow for a 'Real Life' server environment while still developing locally we will want a way to access the VM in our local browser. To do this we need to first set up a project's files. So, get one of your projects (can just be a "Hello World" index.html file if you wish) and place it in the synced directory,

    /home/{your_name}/webserver/sites

To check that the syncing is working: First ensure that you have run the 'trusty' virtual box:

    $ vagrant up 

Then SSH onto it:

    $ vagrant ssh 

You can then navigate to '/var/www/html/' where you should see your local changes reflected.

Next, in the virtual box itself, you will want to set up a virtual host for the above project. This can be done by SSHing onto 'trusty' and creating a virtual host via Apache.

If you need help with how to set up a virtual host, take a look at the help links included at the end of this article. Otherwise, create a virtual host for your project.

Next, you will have to edit your local hosts file (not in the virtual box):

    $ vim /etc/hosts

Add the following line:

    192.168.56.55 {your_website_url}

Notice how we are using the IP we set in the VagrantFile in [Part 2]({% post_url 2015-09-20-vagrant-on-arch-linux-part-2 %}).

Finally, open your browser of choice, and navigate to {your_website_url}. If the above steps were followed correctly you should now see your website as hosted on the Virtual Box. Any changes you make locally will occur in the virtual box, which will then automatically get run through Apache next time you refresh your webpage.

* * *

I hope this guide has been useful to someone. Be sure to read both the Vagrant docs and and related Arch Wiki pages as they cover everything in far greater detail with serveral tips and troubleshooting guides to boot.

Happy coding!

###Further reading:

[Vagrant docs](https://docs.vagrantup.com/v2/)

[Arch Vagrant docs](https://wiki.archlinux.org/index.php/Vagrant)

[Arch VirtualBox docs](https://wiki.archlinux.org/index.php/VirtualBox)
