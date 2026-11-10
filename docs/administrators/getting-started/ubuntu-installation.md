# RemoteLabz's installation guide

!!! warning 
    Do not use the master branch of RemoteLabz, it's too old.

This section guides you through the installation of RemoteLabz and its components (Front and Worker) on an Ubuntu Server 26.04 LTS system. We only supports this version of Ubuntu.

If you have not installed Ubuntu yet, start with [Ubuntu Server 26.04 LTS](https://ubuntu.com/download/server). You can install it from this link [ubuntu-26.04.1-live-server-amd64.iso](https://releases.ubuntu.com/resolute/ubuntu-26.04.1-live-server-amd64.iso).

## Requirements

Only Ubuntu-based distributions are compatible with Remotelabz. This application doesn't work neither in a container, nor in WSL

The first step is to install a ubuntu distro like Ubuntu Server 26.04 LTS.
There is two deployment modes :

- **Single server :** only one computer if you want to use the Front and the Worker on the same server
- **Multi-server :** if you want to separate your Front and your Worker.

!!! note
    The version 2.5 beta for Ubuntu 26.04 LTS Server is available on branch dev from August 21, 2026

### Hardware

To install both the Front and the Worker on the same device, the minimum requirement is 

- a hard disk of at least 30 Go for the system
- 2 Go of RAM
- 1 CPU
- LVM volume management
- 2 logicals volumes for /opt and /var/lib/lxc. The size depends of the maximum number of VM and containers you want to run and image and iso to store.

### Device installed

At the end of the installation, the following devices will be available in RemoteLabz:

| # | Device        | Type      | Operating system       | Status at the end of the installation |
|---|---------------|-----------|------------------------|----------------------------------------|
| 1 | Debian    | container | Debian 11.4            | installed and configured               |
| 2 | Alpine-Stable    | container | Alpine 3.15            | installed and configured               |
| 3 | Ubuntu     | container | Ubuntu Server 24.04 LTS | installed and configured               |
| 4 | Linux Alpine            | VM        | Alpine 3.10            | installed and configured               |
| 5 | "Migration"   | container | Alpine                 | provided by the installation; used to build the DHCP service container (step 7) |
| 6 | "Service"     | container | Alpine + dnsmasq       | **not installed by the script** — created manually, see step 7 |

## Disk partition for the worker

### Storage layout 

![Disk Partition](/images/Administrator/disk_partition.png){ style="display: block; margin: 0 auto;" }

Two **volume groups** (VG) are created on the disk:

| VG | Role | Content |
|---|---|---|
| `ubuntu-vg` | System (Ubuntu default) | LV `ubuntu-lv` ≥ 35 GB → `/` |
| `rlz-vg` | Remotelabz workloads | `lib-lxc`, `rlz-lv`, thin pool `rlz-lxc-pool` |

The `rlz-lxc-pool` thin pool aggregates the space intended for **user LXC instances**. The logical volumes for these instances (`lxc_<uuid>`) are created within it **on the fly**, at the time each instance starts and at the requested size. Without physical pre-allocation, only the space actually written to consumes disk capacity.

### Logical volumes overview

| LV | VG | Type | Size | Mount point | Usage |
|---|---|---|---|---|---|
| `ubuntu-lv` | `ubuntu-vg` | linear | ≥ 35 GB | `/` | Ubuntu system |
| `lib-lxc` | `rlz-vg` | linear | 100 GB (fixed) | `/var/lib/lxc` | Reference LXC templates/containers |
| `rlz-lv` | `rlz-vg` | linear | 60% of remaining free space | `/opt` | QEMU VMs |
| `rlz-lxc-pool` | `rlz-vg` | **thin pool** | all remaining free space | — | Pool for *thin* volumes used by LXC instances |
| `lxc_<uuid>` | `rlz-vg` | thin (within `rlz-lxc-pool`) | size based on flavor, or `DEFAULT_MIN_DISKSPACE_LXC` | `/var/lib/lxc/<uuid>/rootfs` | Rootfs for a user LXC instance |

> Instance LVs (`lxc_<uuid>`) are not part of the initial installation: the worker
> creates and deletes them dynamically (`lxc_setup_disk()` / `lxc_remove_disk()`
> in `InstanceManager`).

!!! info "Partition your disk"
    For example, on a RemoteLabz deploys for 355 users and 570 VM/containers on 2 workers :

    - on the first worker, 193 containers use 289 Go (1.4 Go/container) and 56 VMs use 281 Go (5 Go/VM), respectively
    - on the second worker, 286 containers use 327 Go (1,1 Go/container) and 34 VMs use 284 Go (8,3 Go/VM), respectively

    In this example, all VMs and containers are linux servers.


    !!! note
        On the front, the uploaded iso and VM image are stored in the /opt/remotelabz/public/uploads directory


### Implementation

#### `ubuntu-vg` / `ubuntu-lv` (≥ 35 GB)

These names are the defaults produced by the Ubuntu installer in guided LVM mode. To guarantee at least 35 GB for the system, select the LVM option during partitioning, then adjust the size of the `ubuntu-lv` volume in the installer (`/` ≥ 35 GB).

#### Creating `rlz-vg`, its LVs and the thin pool

```bash
# Create the VG using all the remaining disk space
vgcreate rlz-vg /dev/<partition_or_disk>

# 1) lib-lxc: fixed 100 GB for LXC containers (templates / references)
lvcreate -L 100G -n lib-lxc rlz-vg

# 2) rlz-lv: 60% of the space STILL free (computed after lib-lxc)
lvcreate -l 60%FREE -n rlz-lv rlz-vg

# 3) rlz-lxc-pool: thin pool for LXC instances, using the remaining space.
#    About 5% is kept free in the VG for the pool metadata and for extensions.
lvcreate -l 95%FREE --thinpool rlz-lxc-pool rlz-vg
```

!!!note 
    `-l 60%FREE` and `-l 95%FREE` refer to the free space at the time the command is run.

#### Formatting and mounting

Only the "standard" LVs are formatted and mounted at installation time. Instance LVs are formatted (`mkfs.ext4`) and mounted by the worker when each instance starts.

```bash
mkfs.ext4 /dev/rlz-vg/rlz-lv
mkfs.ext4 /dev/rlz-vg/lib-lxc

mkdir -p /opt /var/lib/lxc
mount /dev/rlz-vg/rlz-lv /opt
mount /dev/rlz-vg/lib-lxc /var/lib/lxc
```

Persistence in `/etc/fstab` (stable identifiers, not the VG name):

```
/dev/mapper/rlz--vg-rlz--lv         /opt            ext4  defaults  0 2
/dev/mapper/rlz--vg-lib--lxc        /var/lib/lxc    ext4  defaults  0 2
```

#### Verification

```bash
vgs                                                    # free/used space per VG
lvs -a -o +lv_size,data_percent,metadata_percent       # LV details and pool usage
df -h /opt /var/lib/lxc                                # actual mounts
```


## Installation of the front
 
### Retrieve the RemoteLabz Front source

Clone the repository in your home directory, as your regular user (do **not** use `sudo git clone`, otherwise the sources become root-owned):

```bash
cd ~
git clone https://github.com/remotelabz/remotelabz.git --branch dev
cd remotelabz
```

You have now a directory `remotelabz` created on your home directory.

!!! warning
    If you want to install a specific version, follow these instructions. For version 2.4.1 for example.
    ```bash    
    git clone https://github.com/remotelabz/remotelabz.git --branch 2.4.1 --single-branch
    ```
    or for version 2.5.0 beta version
    ```bash    
    git clone https://github.com/remotelabz/remotelabz.git --branch dev
    ```

### Run the installation script

```bash
sudo bin/install.sh
```

The installation is interactive. Choose the option that best fits your needs:

![Install Options](/images/Administrator/Install_options.png){ style="display: block; margin: 0 auto;" }

For a complete installation, choose the first option: `Full automatic installation`. This installs all dependencies as well as the RemoteLabz application itself.

If you chose the full automatic installation some answers will be requested by the script:

| Prompt | What to answer |
|--------|----------------|
| **Public IP address** | The script suggests the public IP it detected automatically. If it is correct, type `y`. Otherwise type `n` and enter the correct address. |
| **Deployment type** | Single server (Front and Worker on the same machine): `y`. Two separate servers: `n`. |
| **Configuration file** | The script opens `/opt/remotelabz/.env.local` in your editor. Check the fields listed below, save with `Ctrl+O`, exit with `Ctrl+X`. |
| **Summary** | The script displays a summary of the configuration. If it is correct, type `y`. Otherwise abort, edit `/opt/remotelabz/.env.local` by hand, and re-run the installation. |

The script then performs the remaining steps automatically, including **system configuration** (keyboard language and layout) and the **SSL certificate** setup (you are asked whether the certificate is self-signed).

When the installation is complete, the web interface will be available at `https://<IP_ADDRESS>`.

!!! info
    The default credentials are:

    - Username: `root@localhost`
    - Password: `admin`

!!! warning
    These default credentials are only intended for the initial configuration. **Change them immediately** from the web interface before using RemoteLabz in production.

### Default passwords created by the installer

The installer creates several services with default credentials. You are expected to change them :

| Service  | User             | Default password        |
|----------|------------------|-------------------------|
| MySQL    | `root`           | `RemoteLabz-2022$`      |
| MySQL    | `user`           | `Mysql-Pa33wrd$`        |
| RabbitMQ | `remotelabz-amqp`| `password-amqp`         |
| OpenVPN  | CA key passphrase| `R3mot3!abz-0penVPN-CA2020` |
| Web UI   | `root@localhost` | `admin`                 |

!!! warning
    **Change all of these passwords before putting RemoteLabz in production.** They are documented here because you will need them while the system is still being configured.

#### RabbitMQ and MySQL configurations
The MySQL is configured with the root password : "RemoteLabz-2022\$", and a user "user" is created with password "Mysql-Pa33wrd\$". It is recommended to change it once you have ensured that RemoteLabz is working fine.

!!! Tips
    During the installation process, a `remotelabz-amqp` user is created in RabbitMQ with the password `password-amqp`. If you want to change the password of an existing user `remotelabz-amqp` of your RabbitMQ, you have to type the following command :
    ```
    sudo rabbitmqctl change_password 'remotelabz-amqp' 'new_password'
    ```
    For MySQL, to set the root password to `new_password`
    ```
    sudo mysql -u root -h localhost
    ALTER USER IF EXISTS 'root'@'localhost' IDENTIFIED BY 'new_password';
    FLUSH PRIVILEGES;
    EXITS;
    ```
    The remotelabz default user is `user` and its password `Mysql-Pa33wrd\$`. If you want to change to `new_password` for example, you have to do the following:
    ```
    ALTER USER IF EXISTS 'user'@'localhost' IDENTIFIED BY 'new_password';
    FLUSH PRIVILEGES;
    EXITS;
    ```

!!! info
    During the installation, some actions is done on the directory permission :
    ```bash
    chgrp remotelabz /etc/openvpn/server -R
    chmod g+rx /etc/openvpn/server -R
    ```


### Configure the system mail (Exim4)

RemoteLabz uses the system mailer to send notifications. By default these emails are addressed to `root` and to the local `remotelabz` account, which means they are simply discarded. Configure Exim4 so that all application emails are redirected to a real mailbox you read.

1. Edit `/etc/aliases` and redirect the relevant local accounts to an existing user or to your real address:

    ```bash
    sudo nano /etc/aliases
    ```

    ```bash
    root:      yourname
    remotelabz: yourname
    ```

2. Edit `/etc/exim4/exim4.conf.template` and locate the *REWRITE CONFIGURATION* section. Add one rule per local address so that the message is forwarded to your real mailbox:

    ```bash
    ######################################################################
    #                      REWRITE CONFIGURATION                         #
    ######################################################################

    begin rewrite

    user@* myemail@domain.com FfrsTtcb
    root@* myemail@domain.com FfrsTtcb
    ```

    Replace `user@*` by the local account that sends the mail (e.g. `remotelabz@*`) and `myemail@domain.com` by your real mailbox.

3. Rebuild the Exim4 configuration and restart the service:

    ```bash
    sudo update-exim4.conf
    sudo service exim4 restart
    ```

4. Verify that the addresses are rewritten as expected:

    ```bash
    sudo exim -brw root
    ```

    The command should display `myemail@domain.com` for the rewritten addresses.

---

## Installation of the Worker

### Retrieve the RemoteLabz Worker source 

A `remotelabz-worker` directory will be created in your home directory:

```bash
cd ~
git clone https://github.com/remotelabz/remotelabz-worker.git --branch dev
cd remotelabz-worker
```

!!! tips
    If you want to install only a specific version, you have to do the following instruction, for version 2.4.1 for example.
    ```bash    
    git clone https://github.com/remotelabz/remotelabz-worker.git --branch 2.4.1 --single-branch
    ```
    or for branch Upgrade-2.5
    ```bash    
    git clone https://github.com/remotelabz/remotelabz-worker.git --branch Upgrade-2.5
    ```

### Configure the worker queue

Before running the workers installation, edit the messenger.yaml in the remotelabz-worker directory :

```bash
sudo nano config/packages/messenger.yaml
```

In the section below, replace the default binding key with your Worker's IP:

```yaml
framework:
    messenger:
        transports:
            async: '%env(MESSENGER_TRANSPORT_DSN)%'
            worker:
                dsn: '%env(MESSENGER_TRANSPORT_DSN)%'
                options:
                    queues:
                        messages_worker1:
                            binding_keys: [Worker_1_IP]
```

### Run the installation script

```bash
cd remotelabz-worker
sudo bin/install.sh
```

As with the Front, you will be asked for various pieces of information:

| Prompt | What to answer |
|--------|----------------|
| **Deployment type** | Single server or multi-server configuration. |
| **Network interfaces** | Administration interface (e.g. `enp0s8`). Data interface: press Enter if it is the same as the administration interface. |
| **IP addresses** | Worker IP: in single-server mode, it is identical to the Front's IP. In multi-server mode, enter the Worker's own IP. Also: the interface used for Internet access, and the Front's IP address. |
| **Data network configuration** | Data network, internal data IP and data network gateway. Default values can be kept by pressing Enter. |
| **VPN configuration** | Worker on the same server as the Front: `y`. Otherwise: `n`. |
| **SSH configuration** | Change the password of the `remotelabz-worker` user, or press Enter to keep the default password (then change it later — see step 8). |
| **Secure WebSocket (WSS)** | Type `y` to enable WSS. |
| **Email configuration** | Enter the contact email address, or press Enter to keep the default. |
| **End of configuration** | The `.env.local` file is generated and can be edited manually afterward if needed. |


### Copy certificate files from the front to the worker

Copy the two files `~/EasyRSA/RemoteLabz-WebServer.crt` and `~/EasyRSA/RemoteLabz-WebServer.key` to your **worker** in directory `/opt/remotelabz-worker/config/certs`

#### From the front 
=== "Single server"

    ```bash
    cd ~/EasyRSA
    scp RemoteLabz-WebServer.crt user@127.0.0.1:~
    sudo scp RemoteLabz-WebServer.key user@127.0.0.1:~
    ```

=== "Multi server"

    ```bash
    cd ~/EasyRSA
    scp ~/EasyRSA/RemoteLabz-WebServer.crt user@${WORKER_SERVER}:~
    sudo scp ~/EasyRSA/RemoteLabz-WebServer.key user@${WORKER_SERVER}:~
    ```

#### On the worker

```bash
cd ~
sudo mv RemoteLabz-WebServer.* /opt/remotelabz-worker/config/certs/
sudo sed -i "s/REMOTELABZ_PROXY_USE_WSS=0/REMOTELABZ_PROXY_USE_WSS=1/g" /opt/remotelabz-worker/.env.local
sudo service remotelabz-worker restart
```

!!! warning
    You need to use the same certificate between your front and the worker. Don't forget to copy them and to change it automatically if your certificate expired.


### Start your RemoteLabz Worker service
Normally, the service remotelabz-worker is started during the installation phase and it will start automatically when your system boots up.However, if you need to start the service manually :

```bash
sudo systemctl start remotelabz-worker
```

!!! tips
    To automatically start the service on boot 
    ```bash
    sudo systemctl enable remotelabz-worker
    ```
    To check the status of your service
    ```bash
    sudo service remotelabz-worker status
    ```

You can check the worker's log in `/opt/remotelabz-worker/var/log/prod.log`

!!! warning
    When consuming messages, a timestamp is used to determine which messages the messenger worker is able to consume. Therefore, each machines needs to be time-synchronized. We recommend you to use a service like `ntp` to keep your machines synchronized.

### Add a DHCP Service for your laboratory

Each laboratory has its own network and needs its own DHCP service. RemoteLabz provides a generic container, called **"Migration"**, that you configure once and then export as a reusable device template, called **"Service"**. When you add a "Service" device to a lab, it is automatically configured on that lab's network with the IP **gateway − 1**.

> **Example:** if the network attributed to your lab is `10.10.10.0/27`, the gateway will be `10.10.10.30` and the DHCP service container will receive the IP `10.10.10.29`. The DHCP range below hands out addresses from `10.10.10.10` to `10.10.10.28`, keeping the first addresses and the gateway free.

#### Start the "Migration" device

1. Open the **Sandbox** menu in the web interface.
2. Start the **Migration** device.
3. Open its console. Use the **Show logs** button to display the network parameters attributed to the device.

#### Configure the network inside the container

In the console of the Migration device, set the IP address and the default route that match your lab's network. (Adjust the addresses to your own lab network.)

```bash
ip addr add 10.10.10.29/27 dev eth0
ip route add default via 10.10.10.30
```

#### Install and configure dnsmasq

Next, open the following file:
`vi /etc/systemd/resolved.conf`. In this file, uncomment the DNS= line by removing the # at the beginning, and set it to:
`DNS=1.1.1.1 9.9.9.9`

These are example DNS servers; you can replace them with your preferred DNS servers.

Next, type the following command :
```bash
systemctl restart systemd-resolved
echo 'fr_FR.UTF-8 UTF-8' >> /etc/locale.gen
locale-gen
update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
apt-get update; apt-get -y upgrade; apt-get install -y dnsmasq;
echo "dhcp-range=RANGE_TO_DEFINED" >> /etc/dnsmasq.conf
echo "dhcp-option=3,GW_TO_DEFINED" >> /etc/dnsmasq.conf
echo "server=1.1.1.3" >> /etc/dnsmasq.conf
echo "server=9.9.9.9" >> /etc/dnsmasq.conf
systemctl stop systemd-resolved
systemctl disable systemd-resolved
systemctl disable systemd-networkd
systemctl enable dnsmasq
```

!!! warning
    The command `systemctl disable systemd-networkd` is **mandatory**: otherwise the container will not keep an IP address. This only applies inside the Migration container, not to your host.

Your "Service" device, which is a container, is now ready.

#### Export the container as a device template

1. Stop the Migration device.
2. Click **Export**, type **`Service`** as the new name, and click **Export Device**.
3. In the device menu, **remove the "login" option** from the control protocols: users must not be able to edit this container.
4. From now on, whenever you add a **Service** device to a lab, that lab gets its own DHCP service for all of its devices.
First, stop the "Migration" device. Then, click "Export" and enter "Service" as the New Name. Finally, click the "Export Device" button.


![DHCP Service](/images/Administrator/DHCP-service.png)

#### Leave maintenance mode

The installation is finished and RemoteLabz application must be working now. In order to be fully usable, you'll have to change the parameter in the `/opt/remotelabz/.env.local` according to the following :

```bash
APP_MAINTENANCE=0
```
If you leave this value to 1, nobody, except the administrator will be able to use the application.

---

## Verify the installation

Run through this checklist. When every item is green, your installation is complete.

- [ ] `sudo systemctl status remotelabz` and `sudo systemctl status remotelabz-proxy` are active on the Front.
- [ ] `sudo systemctl status remotelabz-worker` is active on the Worker.
- [ ] The web interface loads at `https://<IP_ADDRESS>`.
- [ ] The Worker appears in the Front's worker list, with its IP (or `127.0.0.1` in single-server mode).
- [ ] A sandbox instance of a provided device (e.g. the Debian container) starts and answers on its console.

- [ ] `timedatectl status` shows synchronized time on every machine.
- [ ] `APP_MAINTENANCE=0` is set in `/opt/remotelabz/.env.local`.

---

## Create your first lab

!!! news
    The tutorial to create a first lab with 1 container and 1 DHCP server : <a href="https://www.youtube.com/watch?v=S0f2-kCIP_k" target="_blank">RemoteLabz first laboratory</a>

### Shibboleth (optional - You have to be registered by Renater)

!!!warning
    You have to activate HTTPS to use Shibboleth authentication method

```bash
cd ~
curl --fail --remote-name https://pkg.switch.ch/switchaai/ubuntu/dists/focal/main/binary-all/misc/switchaai-apt-source_1.0.0~ubuntu20.04.1_all.deb
sudo apt install ./switchaai-apt-source_1.0.0~ubuntu20.04.1_all.deb
sudo apt update
sudo apt install --install-recommends shibboleth
sudo a2enconf shib
sudo a2enmod shib
sudo service apache2 restart
```

Next step, to finish to configure your Shibboleth Service Provider (SP), you have to modify your `/etc/shibboleth/shibboleth2.xml` file, following the guide from Paragraph 4, depend of your Shibboleth Identity Provider (IdP):

 - [SWITCH Shibboleth Service Provider (SP) 3.1 Configuration Guide](https://www.switch.ch/aai/guides/sp/configuration/){target=_blank}
 
RENATER Shibboleth Service has been moved to the official shibboleth site.
 - [Official shibboleth site Installation and Configuration Guide](https://shibboleth.atlassian.net/wiki/spaces/SP3/pages/2065335537/Installation){target=_blank}

You can find all the configuration guides on the following site :

- [On Ubuntu 20.04 LTS](https://www.switch.ch/aai/guides/sp/installation/?os=ubuntu20){target=_blank}

To enable Shibboleth site-wide, you need to change the value of `ENABLE_SHIBBOLETH` environment variable :

```bash
# .env.local
ENABLE_SHIBBOLETH=1
```

## Developer shortcut (sources in the home directory)

If you want to contribute to the project and avoid permission issues with VS Code when the sources live in `/opt`, link the default installation directory to your home directory. 

Front:

```bash
cd ~/remotelabz
sudo cp /opt/remotelabz/.* . -Rf
sudo cp /opt/remotelabz/* . -Rf
sudo rm /opt/remotelabz -Rf
sudo ln -s `pwd` /opt/remotelabz
sudo chown $USER:www-data .* -R
sudo chown remotelabz:www-data * -R
sudo usermod -aG sudo $USER
sudo chmod g+w * -R
```

Worker:

```bash
cd ~/remotelabz-worker
sudo cp /opt/remotelabz-worker/.* . -Rf
sudo cp /opt/remotelabz-worker/* . -Rf
sudo rm /opt/remotelabz-worker -Rf
sudo ln -s `pwd` /opt/remotelabz-worker
sudo chown $USER:www-data .* -R
sudo chown remotelabz-worker:www-data * -R
sudo usermod -aG sudo $USER
sudo chmod g+w * -R
```