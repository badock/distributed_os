# Distributed OpenStack

## Goal

Deploy a distributed OpenStack infrastructure over a decentralized virtual network (n2n). In this scenario, each host of the 
OpenStack infrastructures is virtually connected to the other via two virtual networks (VPN):

* `tap_admin`: administration network used for inter-service communication
* `tap_prod`: production network used to provide public connectivity to hosted virtual machines

The VPN is implemented with [n2n](https://github.com/ntop/n2n), which enables to setup private links in a decentralized way. A virtual network based on n2n can have two kind of nodes:

* `supernodes`: node introduction registry, broadcast conduit and packet relay node for the n2n system. The supernode can service any number of communities and routes packets only between members of the same community.
* `edges nodes`: node daemon for n2n which creates a TAP interface to expose the n2n virtual LAN. On startup n2n creates the TAP interface and configures it then registers with the supernode so it can begin to find other nodes in the community. 

as in the following picture (extracted from [ntop.org](https://www.ntop.org/products/n2n/))

![https://www.ntop.org/wp-content/uploads/2011/08/n2n_network.png](https://www.ntop.org/wp-content/uploads/2011/08/n2n_network.png)

## Installation

### VirtualBox

Download VirtualBox from [https://www.virtualbox.org/](https://www.virtualbox.org/), and install it.

### Vagrant

Download vagrant from the installation page [https://www.vagrantup.com/downloads.html](https://www.vagrantup.com/downloads.html), and install it

## Getting started

### Clone the project and customize the deployment

First clone the project:
```shell script
git remote add origin https://github.com/badock/distributed_os.git
```

### Create a distributed OpenStack

go in the distributed_os folder and create the example infrastructure with one controller and one compute node:

```shell script
cd distributed_os
vagrant up
```

It should may a while to create the infrastructure (40 minutes with 300 MBps connection).

### Play with the infrastructure

#### Connect to the controller and the compute node

Connect to the controller with:
```
vagrant ssh control1
```

Connect to the compute node with:
```
vagrant ssh compute1
```

#### Access the Horizon dashboard

Create a tunnel to the Horizon dashboard with the following `ssh` command:
``` shell script
ssh -L 8000:10.1.0.250:80 192.168.0.42 -l root
```

browsing to [http://localhost:8000](http://localhost:8000) will result in the OpenStack landing page.

Use the credentials located in `/etc/kolla/admin-openrc.sh` on the controller node

### Destroy the distributed OpenStack

Attention: this command will completely erase an existing infrastructure deployed with vagrant

```shell
vagrant destroy --force
```

## Todo

- [ ] VMs cannot ping internet (must set `iptables` rules on controler)
