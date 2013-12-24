# Vagrant

Testing your cloud infrastructure and deployment scripts before you actually provision servers and deploy code is a good idea. [Vagrant](https://www.vagrantup.com/) makes this easy.

In order to use vagrant you will need to have VirtualBox installed. Once you've completed that you'll need to install Vagrant. Note: it is better to run the binary version of Vagrant. If you have a gem version, uninstall it, then []download it](http://www.vagrantup.com/downloads):

You can download and install a default box or you can start with a good `Vagrantfile` and use that to fetch the box you want. In your Rails root create a file called `Vagrantfile`:


    # -*- mode: ruby -*-
    # vi: set ft=ruby :

    Vagrant.configure("2") do |config|
      # By default, I am using ubuntu because I prefer it
      # Lots of people use centos instead
      config.vm.box = "ubuntu"

      # Specifying the url means that if the box is not installed
      # Vagrant knows where to get it. Here I am using a 64-bit
      # version of 12.04 LTS (long-term support)
      config.vm.box_url = "http://cloud-images.ubuntu.com/vagrant/precise/current/precise-server-cloudimg-amd64-vagrant-disk1.box"
      
      # Some basic config for later
      config.vm.network "forwarded_port", guest: 80, host: 8080
      config.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", 2048]
      end
    end

Now, you could have created this with `vagrant init` and that would give you a much better template with all of the options and documentation. Once you've got Vagrant and a Vagrantfile, you are going to need to start the box (this can take a while):

    vagrant up

Great, you should now have a running vagrant box. You can run `vagrant ssh` to test it. If you want to shut that box down:

    vagrant halt

## Bootstrapping and Provisioning

This box is pretty boring at the moment. 

Now we need to prepare the box (which will setup chef and ruby)

