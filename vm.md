---
layout: default
title: Seldon VM
---

# Installing the Seldon Virtual Machine

1. Install **VirtualBox** if not installed already.

    Visit the [download page](https://www.virtualbox.org/wiki/Downloads).  
    Download and follow the instructions.

1. Install **Vagrant** if not installed already.

    Visit the [download page](http://www.vagrantup.com/downloads.html).  
    Download and follow the instructions.

1. Create a directory for the seldon vm vagrant files, and make it the current directory.

        mkdir seldonvm
        cd seldonvm

1. Download the Vagrantfile for the seldon box.

        wget http://static.seldon.io/seldonvm/Vagrantfile

1. Startup the vagrant instance of the box.

        vagrant up

1. Log into the vm.

        vagrant ssh

1. After logging on, see the [Seldon VM usage](vm-usage.html) docs for getting started.

1. For larger datasets you can customize the memory size of the vm.

    In the Vagrantfile change the memory in the following section

        config.vm.provider "virtualbox" do |vb|
            # Use VBoxManage to customize the VM. For example to change memory:
            vb.customize ["modifyvm", :id, "--memory", "4000"]
        end

    Destroy the current instance of the vm and then re-start

        vagrant destroy
        vagrant up


