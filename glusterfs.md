---
layout: default
title: GlusterFS
---

# GlusterFS with Kubernetes

Steps to use [GlusterFS](https://www.gluster.org/) in Kubernetes for Seldon. 

## AWS
Assumes you wish to create your glusterfs cluster in a separate VPC from Kubernetes so its lifetime is not connected to that of the Kubernetes cluster.

 1. [Create an AWS VPC](http://docs.aws.amazon.com/AmazonVPC/latest/GettingStartedGuide/getting-started-create-vpc.html)
    1. Ensure ip range does not overlap with Kubernetes default, e.g. use 192.*
 1. [Create a GlusterFS cluster in VPC](https://www.howtoforge.com/tutorial/high-availability-storage-with-glusterfs-on-debian-8-with-two-nodes/)
    1. Two t2.micro instances minimum
 1. Create a Kubernetes Cluster
 1. [Add glusterfs software for client to each minion](https://www.howtoforge.com/high-availability-storage-with-glusterfs-3.2.x-on-debian-wheezy-automatic-file-replication-mirror-across-two-storage-servers#-setting-up-the-glusterfs-client) and also master if you wish
 1. [Create a VPC Peering connection in glusterFS VPC to Kubernetes VPC](http://docs.aws.amazon.com/AmazonVPC/latest/PeeringGuide/working-with-vpc-peering.html)
    1. Create Peering connection and accept request.
    1. Edit glusterfs routing table to allow traffic to kubernetes
       1. There will be two routing tables. Choose the routing table with the subnet. Add the ip range for kubernetes (usually 172.20.0.0/16). Choose the Peering connection as destination.
    1. Edit kubernetes routing table to allow traffic to glusterfs
       1. There will be two routing tables. Choose the routing table with the subnet. Add the ip range for glusterfs (for example 192.168.0.0/16). Choose the Peering connection as destination.
    1. Update security group for glusterfs inbound to allow kubernetes traffic
    1. Update security group for kubernetes inbound to allow glusterfs traffic
 1. Test mount glusterfs volume on master or minion node, e.g. 
    ```mkdir /mnt/glusterfs```,
    ```mount.glusterfs 192.168.0.149:/gv0 /mnt/glusterfs```
 1. Ensure you follow docs for [glusterfs use in Seldon](install.html#storage)

## Other Cloud Providers

Contributions welcome.
