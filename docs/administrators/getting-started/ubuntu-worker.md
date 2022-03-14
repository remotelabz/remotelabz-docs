# RemoteLabz's Worker installation guide

This section guides you through the installation of RemoteLabz's worker and its components on an Ubuntu system. We assume you have already installed an Ubuntu Server 20.04 LTS.

!!! note "Foreword"

    - This section has only been tested with **Ubuntu Server 20.04 LTS**

# Retrieve the RemoteLabz source
A remotelabz directory will be create on your home directory.
```bash
cd ~
git clone https://github.com/crestic-urca/remotelabz-worker.git
cd remotelabz-worker
```

# Installation
You should modify the `.env` file according to your environment

``` bash
nano .env
# you may change the network interface name
# DATA_INTERFACE will be used by the virtual machine
# ADM_INTERFACE will be used by the RemoteLabz's front and the RabbitMQ to communicate with the worker. This interface is also used to ssh connexion
DATA_INTERFACE="enp0s8"
ADM_INTERFACE="enp0s3"
# you may change the MESSENGER_TRANSPORT_DSN variable with the following, with your credentials, and the RabbitMQ IP or its FQDN
MESSENGER_TRANSPORT_DSN=amqp://remotelabz-amqp:password-amqp@X.X.X.X:5672/%2f/messages
sudo ./install
```
# Configuration
### Activate the forward between the interface
In the file `/etc/sysctl.conf`, looking for the line `#net.ipv4.ip_forward=1` and uncomment it. Then, reload this `/etc/sysctl.conf` file
```bash
sudo sysctl --system
```

### Configure the route from the worker to the front for the VM's network
We assume you have configured now all variables in your .env.local, according the .env.local of your front, which was modified after a copy of the .env.dist
```bash
source /opt/remotelabz-worker/.env.local
sudo ip route add $VPN_NETWORK/$VPN_NETWORK_NETMASK via $FRONT_IP_ADDRESS
```

### Internet access (From Version 2.3.0 and after)
For testing the Internet access on RemoteLabz Version 2.3.0 and after, you have to enable the NAT with the command
```bash
sudo iptables -t nat -A POSTROUTING -o $ADM_INTERFACE -s $LAB_NETWORK -j MASQUERADE
```

In the next version (2.5.0), Internet connection will be configurable from the graphical interface and the NAT will be like that :
```bash
sudo iptables -t nat -A POSTROUTING -o $BRIDGE_INT -s $LAB_NETWORK -j MASQUERADE
```

### Instances

```bash
sudo service remotelabz-worker start
```

You can check the log of the worker in `~/remotelabz-worker/var/log/prod.log`

## Options

- `-p` Port used by remotelabz-worker
    - Default : `8080`

!!! warning
    When consuming messages, a timestamp is used to determine which messages the messenger worker is able to consume. Therefore, each machines needs to be time-synchronized. We recommand you to use a service like `ntp` to keep your machines synchronized.

## In progress (in next version 2.4.1)

To install and test a service container :
Last Debian stable (16 Nov. 2021) Linux Install
```
sudo lxc-create -t download -n Container_Name -- -d debian -r bullseye -a amd64 --keyserver hkp://keyserver.ubuntu.com
```
Edit the config of the container:
```
sudo nano /var/lib/lxc/Container_Name/config
```

## LXD In progress (in next version 2.4.1)
```
sudo lxd init
```

Answer to all questions
```
Would you like to use LXD clustering? (yes/no) [default=no]: 
Do you want to configure a new storage pool? (yes/no) [default=yes]: 
Name of the new storage pool [default=default]: 
Name of the storage backend to use (dir, lvm, zfs, ceph, btrfs) [default=zfs]: 
Create a new ZFS pool? (yes/no) [default=yes]: 
Would you like to use an existing empty block device (e.g. a disk or partition)? (yes/no) [default=no]: 
Size in GB of the new loop device (1GB minimum) [default=15GB]: 
Would you like to connect to a MAAS server? (yes/no) [default=no]: 
Would you like to create a new local network bridge? (yes/no) [default=yes]: no
Would you like to configure LXD to use an existing bridge or host interface? (yes/no) [default=no]: 
Would you like the LXD server to be available over the network? (yes/no) [default=no]: 
Would you like stale cached images to be updated automatically? (yes/no) [default=yes] 
Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]: 
```

To list available image to install :
```
lxc image list ubuntu:20.04
```

Install the Service LXC container
```
lxc launch ubuntu:20.10 Service
```

Create the bridge 
```
sudo ovs-vsctl add-br test
```
Create a tap
```
sudo ip tuntap add tap1 mode tap
```

Affect a key to a profile
lxc profile device get RemoteLabz eth0 parent
lxc profile device set RemoteLabz eth0 parent test

---
Modify the config of a container named 'Service'

lxc config device add Service eth0 nic nictype=routed


lxc config device add Service eth0 nic nictype=bridged parent=test
lxc config device remove Service eth0

lxc network create test2 --type=bridge ipv4.address=10.10.10.0/24 ipv6.address=none
lxc network attach testbr Service eth0 eth0
lxc config device set Service eth0 ipv4.address 10.10.10.2

lxc config device unset Service eth0 ipv4.address
----
Créer le bridge
sudo ovs-vsctl add-br test-br
lxc copy Service New-lxc
lxc config device add New-lxc eth0 nic nictype=bridged parent=test-br
lxc start New-lxc
lxc exec New-lxc ip addr add 10.10.10.2/24 dev eth0

--
Config par defaut de Service:
 lxc config show Service
architecture: x86_64
config:
  image.architecture: amd64
  image.description: ubuntu 20.10 amd64 (release) (20210720)
  image.label: release
  image.os: ubuntu
  image.release: groovy
  image.serial: "20210720"
  image.type: squashfs
  image.version: "20.10"
  volatile.base_image: 96234c60d65202ed8ce934f05311f3ec41db78fae4d5662c05ea390bedddcc62
  volatile.idmap.base: "0"
  volatile.idmap.current: '[{"Isuid":true,"Isgid":false,"Hostid":1000000,"Nsid":0,"Maprange":1000000000},{"Isuid":false,"Isgid":true,"Hostid":1000000,"Nsid":0,"Maprange":1000000000}]'
  volatile.idmap.next: '[{"Isuid":true,"Isgid":false,"Hostid":1000000,"Nsid":0,"Maprange":1000000000},{"Isuid":false,"Isgid":true,"Hostid":1000000,"Nsid":0,"Maprange":1000000000}]'
  volatile.last_state.idmap: '[{"Isuid":true,"Isgid":false,"Hostid":1000000,"Nsid":0,"Maprange":1000000000},{"Isuid":false,"Isgid":true,"Hostid":1000000,"Nsid":0,"Maprange":1000000000}]'
  volatile.last_state.power: STOPPED
  volatile.uuid: f169c376-a653-496c-a923-02275f0785ea
devices:
  root:
    path: /
    pool: default
    type: disk
ephemeral: false
profiles: []
stateful: false
description: ""

https://linuxcontainers.org/lxd/getting-started-cli/