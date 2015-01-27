---
layout: default
title: Seldon VM
---

# Seldon Virtual Machine

Seldon will shortly be releasing a virtual machine with all services pre-wired for you to test with your service data and MovieLens Demo. To receive this please [sign up for the beta](http://www.seldon.io/open-source)

##Instructions for installing the vm

1. Install **VirtualBox** if not installed already.

    Visit the [download page](https://www.virtualbox.org/wiki/Downloads).  
    Download and follow the instructions.

1. Install **Vagrant** if not installed already.

    Visit the [download page](http://www.vagrantup.com/downloads.html).  
    Download and follow the instructions.

1. Create a directory for the seldon vm vagrant files, and make it the current dir

        mkdir seldonvm
        cd seldonvm

1. Dowload the Vagrantfile for the seldon box

        wget https://s3-eu-west-1.amazonaws.com/...

1. Startup the vagrant instance of the box

        vagrant up

1. Log onto the instance

        vagrant ssh

1. From inside the vm, navigate to the seldon startup dir and launch the seldon services.

        cd ~/seldon/dist
        ./start-all


