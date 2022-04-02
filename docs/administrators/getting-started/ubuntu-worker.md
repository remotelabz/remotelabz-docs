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

!!! warning
    If you want to install only a specific version, you have to do the following instruction, for version 2.4.1 for example.
    ```bash    
    git clone https://github.com/crestic-urca/remotelabz-worker.git --branch 2.4.1 --single-branch
    ```
    ou
    ```bash    
    git clone https://github.com/crestic-urca/remotelabz-worker.git --branch dev
    ```

# Installation
You should modify the `.env` file according to your environment

``` bash
nano .env
# you may change the network interface name
# DATA_INTERFACE will be used by the virtual machine
# ADM_INTERFACE will be used by the RemoteLabz's front and the RabbitMQ to communicate with the worker. This interface is also used to ssh connexion
DATA_INTERFACE="enpX"
ADM_INTERFACE="enpY"
#The DATA_INTERFACE will be bridge, during the install phase, with the br-worker-data interface (which will be an OVS)
#Must be equal to the BASE_NETWORK of your .env.local on your front
LAB_NETWORK=10.0.0.0/8

# you may change the MESSENGER_TRANSPORT_DSN variable with the following, with your credentials, and the RabbitMQ IP or its FQDN. If the RabbitMQ is on your front, you have to use the adm network 
MESSENGER_TRANSPORT_DSN=amqp://remotelabz-amqp:password-amqp@X.X.X.X:5672/%2f/messages
#Must be equal to the value of the parameter server in your /etc/openvpn/server/server.conf on your Front
VPN_NETWORK=X.X.X.X
VPN_NETWORK_NETMASK=255.255.255.0
#The FRONT_DATA must be equal to the IP of the PUBLIC_ADDRESS parameter of the .env.local from the Front 
FRONT_DATA_IP=Y.Y.Y.Y

sudo ./install
sudo cp .env /opt/remotelabz-worker/.env.local
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
sudo ip route add $VPN_NETWORK/$VPN_NETWORK_NETMASK via $FRONT_DATA_IP
```

### Configure WSS
If you have configured the front with HTTPS, you have to do the following steps:
```bash
cp ~/RemoteLabz-WebServer.* /opt/remotelabz-worker/config/certs/
```
Modify your `.env.local` to have the next line :
```bash
REMOTELABZ_PROXY_USE_WSS=1
REMOTELABZ_PROXY_SSL_KEY="/opt/remotelabz-worker/config/certs/RemoteLabz-WebServer.key"
#If intermediate certificate exist, you have to paste the cert and the intermediate in the same .pem file
REMOTELABZ_PROXY_SSL_CERT="/opt/remotelabz-worker/config/certs/RemoteLabz-WebServer.crt"
```

!!! warning
    You need to use the same certification between your front and this worker. Don't forget to copy them and to change it automatically when your certificate expired.



### Internet access (From Version 2.3.0 and before 2.4.0)
For testing the Internet access on RemoteLabz Version 2.3.0 to 2.4.0, you have to enable the NAT with the command
```bash
sudo iptables -t nat -A POSTROUTING -o $ADM_INTERFACE -s $LAB_NETWORK -j MASQUERADE
```
From version 2.4.0, this command is executed with the remotelabz-worker service. The admin network interface is used to nat the network because we assume the admin network is connected to Internet. For security reason, we recommand to use the data network to route and nat the data VM. So, your data network need to be connect to internet.

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