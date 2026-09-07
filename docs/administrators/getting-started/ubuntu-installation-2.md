# RemoteLabz's installation guide

This section guides you through the installation of RemoteLabz and its components on an Ubuntu system. We assume you have already installed an Ubuntu Server 26.04 LTS. For now, we support only this version of Ubuntu.

## Requirements

Only Ubuntu-based distributions are compatible with Remotelabz.

The first step is to install a ubuntu distro like Ubuntu Server 26.04 LTS 
https://releases.ubuntu.com/resolute/ubuntu-26.04.1-live-server-amd64.iso
on

- only one computer if you want to use the Front and the Worker on the same server
- 2 computers if you want to separate your Front and your Worker.

!!! warning
    This application doesn't work neither in a container, nor in WSL

To install both the Front and the Worker on the same device, the minimum requirement is 

- a hard disk of at least 30 Go.
- 2 Go of RAM
- 1 CPU

This depends of the number of VMs, containers, and, operating system used, you want to run simultaneously. At the end of the installation, 4 devices will be installed and configured :

- 3 containers with Debian 11.4, Alpine 3.15, Ubuntu Server 24.04 LTS
- 1 VM Alpine 3.10

The 5th device, called "Migration" is another Alpine used for configuration.At the end of the installation, a 6th container with a DHCP service must be created.


### Configure the mail (Exim4)
1. Configure the /etc/aliases to redirect all mail to root to an existing user of your OS
2. Check the aliases with the command `exim -brw root`
3. Edit the file `/etc/exim4/exim4.conf.template` and locate the part "Rewrite configuration" until you have an output like the following example :
```bash
######################################################################
#                      REWRITE CONFIGURATION                         #
######################################################################

begin rewrite

user@* myemail@domain.com FfrsTtcb
root@* myemail@domain.com FfrsTtcb
```
4. Update your exim configuration with command `sudo update-exim4.conf`, following the command `sudo service exim4 restart`
5. Check if all addresses are rewritten with the command `exim -brw root`

## Install of the Front

### Retrieve the RemoteLabz Front source

To start with, clone the repository in the user's home directory and run the installation script : 

``` bash
cd ~
sudo git clone https://github.com/remotelabz/remotelabz.git --branch dev
cd remotelabz
sudo bin/install.sh
```

The installation is automated: choose the option that best fits your needs.

![Install Options](/images/Administrator/Install_options.png){ style="display: block; margin: 0 auto;" }

For a complete installation, choose the first option: `Full automatic installation`. This will install all dependencies as well as the Remotelabz application itself.

### Step for the installation script
You will be asked for various pieces of information, as follows:

- **Public IP address** : The script suggests a public IP address that it has detected automatically.
    - If the suggested IP is correct: type y.
    - Otherwise: type n, then enter the correct IP address when prompted.
- **Deployment type** Specify whether the deployment will be on a single server (Front and Worker on the same machine) or on two separate machines.
    - Single server: answer y.
    - Deployment on two separate servers: answer n.
- **Configuration file** The script then opens a configuration file for editing. Pay particular attention to the following fields:
``` bash
PUBLIC_ADDRESS
IP_ADDRESS
MySQL credentials
SSL_CA_KEY_PASSPHRASE
CONTACT_MAIL
```
To exit the editor and save: Ctrl+O.

- **Summary** A summary of the configuration is displayed.
    - If everything looks correct: enter y.
    - If there is an issue with the current configuration, you can manually edit `/opt/remotelabz/.env.local` and run the installation again.

- **Installation of the application**
    - Step 1 — System configuration Configure the keyboard language and layout.
    - Step 4 — SSL certificate Specify whether the SSL certificate being used is self-signed.
- **End of installation** Once the installation is complete, the web interface can be accessed from the server's IP address, for example: `https://IP_ADDRESS`

You can now test your RemoteLabz front with your internet navigator but you will just make connection until the worker is not installed.

!!! info
    The default credentials are :

    - Username : `root@localhost`
    - Password : `admin`

    You may change those values by using the web interface.


!!! warning
    When consuming messages, a timestamp is used to determine which messages the messenger worker is able to consume. Therefore, each machines needs to be time-synchronized. We recommend you to use a service like `ntp` to keep your machines synchronized.

!!! warning
    Now you have to install RemoteLabz Worker

#### RabbitMQ and MySQL pre-configurations
The MySQL is configured with the root password : "RemoteLabz-2022\$", and a user "user" is created with password "Mysql-Pa33wrd\$". It is recommended to change it once you have ensured that RemoteLabz is working fine.

!!! Tips
    During the `install_requirement.sh` process, a `remotelabz-amqp` user is created in RabbitMQ with the password `password-amqp`. If you want to change the password of an existing user `remotelabz-amqp` of your RabbitMQ, you have to type the following command :
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

#### OpenVPN pre-configuration
The default passphrase used during the `install.sh` process is `R3mot3!abz-0penVPN-CA2020`. You can find this value in your `.env` file

```bash
SSL_CA_KEY_PASSPHRASE="R3mot3!abz-0penVPN-CA2020"
```
If you decided to change it during the installation process, don't forget to do this in the `/opt/remotelabz/.env.local`.

!!! warning
    The last line `push "route 10.11.0.0 255.255.0.0"` in your `/etc/openvpn/server/server.conf` must be modified if you modify, in your `.env.local` file, the parameters of the two next lines 
    ```BASE_NETWORK=10.11.0.0
    BASE_NETWORK_NETMASK=255.255.0.0```
    This network will be the network used for your laboratory. Your user must have a route on its workstation to join, via his VPN, his laboratory. Be careful, this network must be different of your home network.


!!! Tips 
    For developper, if you want to contribute to the project and not have any rights issues with VScode and your sources installed on your home, you have to link the default installation directory to your remotelabz directory in your home.

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


## Installation of the Worker

### Retrieve the RemoteLabz Worker source 
A remotelabz directory will be created on your home directory.
```bash
cd ~
git clone https://github.com/remotelabz/remotelabz-worker.git --branch dev
cd remotelabz-worker
sudo bin/install.sh
```

!!! tips
    If you want to install only a specific version, you have to do the following instruction, for version 2.4.1 for example.
    For Ubuntu 26 only the dev branch is compatible.
    ```bash    
    git clone https://github.com/remotelabz/remotelabz-worker.git --branch 2.4.1 --single-branch
    ```
    or
    ```bash    
    git clone https://github.com/remotelabz/remotelabz-worker.git --branch dev
    ```

### Step of the installation script
As with the front, you will be asked for various pieces of information : 

- **Deployment type** Specify whether the deployment is on a single server or in a multi-server configuration.
- **Network interfaces**
    - Administration network interface (example: `enp0s8`).
    - Data network interface. If it is the same as the administration network, simply press Enter (example: `enp0s8`).
- **IP addresses**
    - Worker IP: identical to the Front's IP in "single server" mode, otherwise enter the Worker's own IP.
    - Interface used for Internet access.
    - Front's IP address.
- **Data network configuration**
  Set the data network, internal data IP, and data network gateway (default values can be kept by pressing Enter).
- **VPN configuration**
    - If the Worker is deployed on the same server as the Front: answer `y`.
    - Otherwise: answer `n`.
- **SSH configuration**
    - Change the password for the `remotelabz-worker` user.
    - Or keep the default password by pressing Enter.
- **Secure WebSocket (WSS)**
    - Type `y` to enable WSS.
- **Email configuration**
    - Enter the contact email address.
    - Or keep the default email by pressing Enter.
- **End of configuration** The `.env.local` file is generated and can be edited manually afterward if needed.


!!! tips
    For developper, if you want to contribute to the project and not have any rights issues with VScode and your sources installed on your home, you have to link the default installation directory to your remotelabz directory in your home.
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

### Configuration of the Worker

Before adding the worker's IP on the front, be sure to modify the `messenger.yaml` file. On your worker, in the file `/opt/remotelabz-worker/config/packages/messenger.yaml`, you have the following lines. Change the default IP by your worker's IP.

```bash
    messages_worker1:
        binding_keys: [Worker_1_IP]
```
in the following section
```bash
framework:
    messenger:
        transports:
            async: '%env(MESSENGER_TRANSPORT_DSN)%'
            worker: 
                dsn: '%env(MESSENGER_TRANSPORT_DSN)%'
                options:
                    queues:
```

If you add another worker, you will have another additional lines,
```bash
    messages_worker2:
        binding_keys: [Worker_2_IP]
```
and so on.

You can now configure the worker IP on the web interface of the front by clicking on the button + and type its IP. If you use only worker is on the same server as the front, you can put 127.0.0.1 .

![Add a worker](/images/Administrator/AddWorker.png)

You can check the worker's log in `/opt/remotelabz-worker/var/log/prod.log`

!!! warning
    When consuming messages, a timestamp is used to determine which messages the messenger worker is able to consume. Therefore, each machines needs to be time-synchronized. We recommend you to use a service like `ntp` to keep your machines synchronized.

If you have an error 500, do the following :
```bash
cd /opt/remotelabz
sudo chown -R www-data:www-data config/jwt
sudo chown -R www-data:www-data var
```

## Worker-Side Configuration for SSL Certificates

Once the front and the worker are installed, SSL certificates generated on the Front must be copied to the Worker server.
To copy the certificates from the front :
```bash
cd ~/EasyRSA
scp RemoteLabz-WebServer.crt user@127.0.0.1:~
scp RemoteLabz-WebServer.key user@127.0.0.1:~
```
And on the worker : 
```bash
sudo mv RemoteLabz-WebServer.* /opt/remotelabz-worker/config/certs/
sed -i 's/REMOTELABZ_PROXY_USE_WSS=0/REMOTELABZ_PROXY_USE_WSS=1/g' /opt/remotelabz-worker/.env.local
sudo systemctl restart remotelabz-worker
```


### Add a DHCP Service for your laboratory
In the device list, you will find a device with the name "Migration". This container will be used to configure, via the Sandbox function, a new container, called "Service" to provide a DHCP service to your laboratory. Each laboratory has its own DHCP service and its own network, so the RemoteLabz needs to configure this generic container to offer IP on the right network. For each lab, if you add the DHCP service container, it will be configured with the IP : IP_Gateway - 1. 
For example, if your attributed network is 10.10.10.0/27, your gateway will be 10.10.10.30 and you DHCP service container will have the IP 10.10.10.29 .

First : go to the sandbox menu and start the "Migration" device. Next, in the console of the started device, configure the network of the device (show the log, with "Show logs" button to know it) 

!!! tips
    Add an IP address `ip addr add X.X.X.X/M dev eth0`

    Add the default route `ip route add default via X.X.X.X`


Next, type the following command :
```bash
sudo rm /etc/resolv.conf
echo "nameserver 1.1.1.1" > /etc/resolv.conf
apt-get update; apt-get -y upgrade; apt-get install -y dnsmasq;
echo "dhcp-range=RANGE_TO_DEFINED" >> /etc/dnsmasq.conf
echo "dhcp-option=3,GW_TO_DEFINED" >> /etc/dnsmasq.conf
systemctl stop systemd-resolved
systemctl disable systemd-resolved
systemctl disable systemd-networkd
systemctl enable dnsmasq
```

The line (`systemctl disable systemd-networkd`) is mandatory otherwise your container will not have any IP.

Your "Service" device, which is a container, is now ready. You have to stop the Migration device, click on Export and type, as a New Name : Service and click on the button "Export Device"
On your lab, if you add Service device, you will have a DHCP service for all your devices of your lab.
In the device menu, remove the "login" option from the control protocols, as users should not edit this VM.

![DHCP Service](/images/Administrator/DHCP-service.png)

The installation is finished and RemoteLabz application must be working now. In order to be fully usable, you'll have to change the parameter in the `/opt/remotelabz/.env.local` according to the following :

```bash
APP_MAINTENANCE=0
```
If you leave this value to 1, nobody, except the administrator will be able to use the application.

## Create your first lab

!!! news
    The tutorial to create a first lab with 1 container and 1 DHCP server : <a href="https://www.youtube.com/watch?v=S0f2-kCIP_k" target="_blank">RemoteLabz first laboratory</a>


