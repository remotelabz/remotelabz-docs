# RemoteLabz's installation guide

This section guides you through the installation of RemoteLabz and its components on an Ubuntu system. We assume you have already installed an Ubuntu Server 20.04 LTS. For now, we support only this version of Ubuntu.

## Requirements

Only Ubuntu-based distributions are compatible with Remotelabz.

The first step is to install a ubuntu distro like Ubuntu Server 24.04 LTS on

- only one computer if you want to use the Front and the Worker on the same server
- 2 computers if you want to separate your Front and your Worker.

!!! note
    The future version 2.5 for Ubuntu 24.04 LTS Server is available on branch Upgrade-2.5

!!! warning
    This application doesn't work neither in a container, nor in WSL

To install both the Front and the Worker on the same device, the minimum requirement is 

- a hard disk of at least 30 Go for the system
- 2 Go of RAM
- 1 CPU
- LVM volume management
- 2 logicals volumes for /opt and /var/lib/lxc. The size depends of the maximum number of VM and containers you want to run and image and iso to store.

At the end of the installation, 4 devices will be installed and configured :

- 3 containers with Debian 11.4, Alpine 3.15, Ubuntu Server 20.04 LTS
- 1 VM Alpine 3.10

The 5th device, called "Migration" is another Alpine used for configuration.At the end of the installation, a 6th container with a DHCP service must be created.

!!! warning "Requirements"
    RemoteLabz require PHP 8.4 to work properly.

!!! info "Partition your disk"
    For the worker, the image and iso of each virtual device is stored in `/opt/remotelabz-worker/images`, `/opt/remotelabz-worker/iso`, respectively. Each laboratory stores the user's VM in `/opt/remotelabz-worker/instances` and the container in the lxc default working directory `/var/lib/lxc`
    
    To avoid storage problem on the system, we recommand to build 2 Logical Volumes in the Volume named rlz-vg for :
    
    - `/opt`
    - `/var/lib/lxc`


!!! question "How size my partition ?"
    For example, on a RemoteLabz deploys for 355 users and 570 VM/containers on 2 workers :

    - on the first worker, 193 containers use 289 Go (1.4 Go/container) and 56 VMs use 281 Go (5 Go/VM), respectively
    - on the second worker, 286 containers use 327 Go (1,1 Go/container) and 34 VMs use 284 Go (8,3 Go/VM), respectively

    All VMs and containers are linux servers.

    For the system, we need at :
    
    - on the front, at least 30 Go + space to store the uploaded iso and VM images

    !!! note
        On the front, the uploaded iso and VM image are stored in the /opt/remotelabz/public/uploads directory

    - on the worker, at least 35 Go for Linux system and the rest for a Volume group which have to name "rlz-vg". In this volume, you have to create one logical volume for the /opt directory which contains all user instances of VM (Qemu). We recommand to use 60% of your free space for /opt and let the 40% which is automatically use by the container.


## Installation of the requirements

!!! news
    The tutorial of this part : <a href="https://www.youtube.com/watch?v=tc-X8UDvuss" target="_blank">RemoteLabz Installation Part1</a>


### Retrieve the RemoteLabz Front source
A remotelabz directory will be created on your home directory.
```bash
cd ~
git clone https://github.com/remotelabz/remotelabz.git --branch master
```

You have now a directory `remotelabz` created on your home directory.

!!! warning
    If you want to install a specific version, follow these instructions. For version 2.4.1 for example.
    ```bash    
    git clone https://github.com/remotelabz/remotelabz.git --branch 2.4.1 --single-branch
    ```
    or for Upgrade-2.5 version
    ```bash    
    git clone https://github.com/remotelabz/remotelabz.git --branch Upgrade-2.5
    ```

### Install the requirements
```bash
cd remotelabz
sudo ./bin/install_requirement.sh
```

After this process, you have to take into account the following information :

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
    ALTER USER IF EXISTS 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'new_password';
    FLUSH PRIVILEGES;
    EXIT;
    ```
    The remotelabz default user is `user` and its password `Mysql-Pa33wrd\$`. If you want to change to `new_password` for example, you have to do the following:
    ```
    ALTER USER IF EXISTS 'user'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'new_password';
    FLUSH PRIVILEGES;
    EXIT;
    ```

#### OpenVPN pre-configuration
The default passphrase used during the `install_requirement.sh` process is `R3mot3!abz-0penVPN-CA2020`. You can find this value in your `.env` file

```bash
SSL_CA_KEY_PASSPHRASE="R3mot3!abz-0penVPN-CA2020"
```
If you decide to change it, don't forget to do this in the `/opt/remotelabz/.env.local`.

!!! warning
    The last line `push "route 10.11.0.0 255.255.0.0"` in your `/etc/openvpn/server/server.conf` must be modified if you modify, in your `.env.local` file, the parameters of the two next lines 
    ```BASE_NETWORK=10.11.0.0
    BASE_NETWORK_NETMASK=255.255.0.0```
    This network will be the network used for your laboratory. Your user must have a route on its workstation to join, via his VPN, his laboratory. Be careful, this network must be different of your home network.

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

!!! news
    The tutorial of this part : <a href="https://www.youtube.com/watch?v=9Kzn3Z65Avk" target="_blank">RemoteLabz Installation Part2</a>

The install process will create the directory `/opt/remotelabz`.

While you're in RemoteLabz root directory :

``` bash
cd ~/remotelabz
sudo ./bin/install
```
The install process may take around 5 minutes

After this process, you have to take into account the following information :

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
The default passphrase used during the `install_requirement.sh` process is `R3mot3!abz-0penVPN-CA2020`. You can find this value in your `.env` file

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


#### Configure the RemoteLabz database
Run the `remotelabz-ctl` configuration utility to setup your database :

```bash
cd /opt/remotelabz
sudo remotelabz-ctl reconfigure database
```

Don't forget to edit your `/opt/remotelabz/.env.local` :

!!! warning
    Don't forget to modify the line `PUBLIC_ADDRESS="your-url-or-ip-of-your-front"`

#### Generate API keys
At the root of your RemoteLabz directory:

```bash
cd /opt/remotelabz
sudo mkdir -p config/jwt
sudo openssl genpkey -out config/jwt/private.pem -aes256 -algorithm rsa -pkeyopt rsa_keygen_bits:4096
#Your can use as passphrase "JWTTok3n"
sudo openssl pkey -in config/jwt/private.pem -out config/jwt/public.pem -pubout
sudo chown remotelabz:www-data * -R
sudo chmod g+w /opt/remotelabz/var -R
sudo chmod g+w /opt/remotelabz/public/uploads -R
sudo chmod g+w config/templates
sudo chmod g+r config/jwt/private.pem

# Replace 'yourpassphrase' by your actual passphrase
echo "JWT_PASSPHRASE=\"JWTTok3n\"" | sudo tee -a .env.local
```

!!! warning
    Avoid special character in the JWT, otherwise you will have some errors

!!! warning
    You have to configure the variable DEPLOY_SINGLE_SERVER in your .env.local . Set to the value 1 if you deploy the RemoteLabz on only one server and set to 0 otherwise. The default value is 1 because we assume you deploy the front and the worker on the same server.


!!! tips
    In order for the app to work correctly, a key pair is created for JWT. You can find detailed configuration in [the LexikJWTAuthenticationBundle doc](https://github.com/lexik/LexikJWTAuthenticationBundle/blob/master/Resources/doc/index.rst#generate-the-ssh-keys).


### Configure RemoteLabz to use SSL

This section guides you through the configuration of SSL between all service of the RemoteLabz.

#### Configure your Apache 2 with HTTPS

During the installation process, the file `200-remotelabz-ssl.conf` is copied in your `/etc/apache2/sites-available` directory. The certificate is defined as follow :
```bash
        SSLCertificateFile	/etc/apache2/RemoteLabz-WebServer.crt
        #SSLCertificateChainFile /etc/ssl/certs/remotelabz._INTERMEDIATE.cer
        SSLCertificateKeyFile /etc/apache2/RemoteLabz-WebServer.key
```

Two case, either you have an official certificate or you have to generate your own certificate.
##### Official certificate

If you have an official certificate, you have to copy it in your `/etc/apache2` directory and rename it to `RemoteLabz-WebServer.crt` and `RemoteLabz-WebServer.key`. Next, you have to activate this site:
```bash
sudo a2ensite 200-remotelabz-ssl.conf
sudo a2enmod ssl
sudo service apache2 reload
```

##### Self-signed certificate
Execute the script 
```bash
cd ~
sudo remotelabz/bin/install_ssl.sh
```

!!! tips
    You can verify your certificate with the following command : 
    ```bash
    openssl x509 -noout -text -in /etc/apache2/RemoteLabz-WebServer.crt
    ```

!!! warning 
    Don't forget to reload the Apache 2 service
    ```bash
    sudo systemctl restart apache2
    ```

#### Start the RemoteLabz Front

In order to be able to control instances on [the worker](https://gitlab.remotelabz.com/remotelabz/remotelabz-worker), you need to start **Symfony Messenger** :

```bash
sudo systemctl enable remotelabz
sudo systemctl enable remotelabz-proxy
sudo systemctl enable remotelabz-route-monitor
sudo systemctl enable remotelabz-clean-notification
sudo systemctl enable remotelabz-git-version-update

sudo systemctl start remotelabz
sudo systemctl start remotelabz-proxy
sudo systemctl start remotelabz-route-monitor
sudo systemctl start remotelabz-clean-notification
sudo systemctl start remotelabz-git-version-update
```

You can now test your RemoteLabz front with your internet navigator but you will just make connection until the worker is not installed.

!!! warning
    The front is only listen on TCP/443

!!! info
    The url : https://Your_IP
    The default credentials are :

    - Username : `root@localhost`
    - Password : `admin`

    You may change those values by using the web interface.


!!! warning
    When consuming messages, a timestamp is used to determine which messages the messenger worker is able to consume. Therefore, each machines needs to be time-synchronized. We recommend you to use a service like `ntp` to keep your machines synchronized.

!!! warning
    Now you have to install RemoteLabz Worker

## Installation of the Worker

!!! news
    The tutorial of this part : <a href="https://www.youtube.com/watch?v=8xgBZaWVvQk" target="_blank">RemoteLabz Installation Part3</a>

### Retrieve the RemoteLabz Worker source 
A remotelabz directory will be created on your home directory.
```bash
cd ~
git clone https://github.com/remotelabz/remotelabz-worker.git --branch master
```
A `remotelabz-worker` directory is created after the previous command.

!!! tips
    If you want to install only a specific version, you have to do the following instruction, for version 2.4.1 for example.
    ```bash    
    git clone https://github.com/remotelabz/remotelabz-worker.git --branch 2.4.1 --single-branch
    ```
    or for branch Upgrade-2.5
    ```bash    
    git clone https://github.com/remotelabz/remotelabz-worker.git --branch Upgrade-2.5
    ```

### Installation of the RemoteLabz worker application
```bash
cd ~/remotelabz-worker
cp .env .env.local
```

You should modify the `~/remotelabz-worker/.env.local` file according to your environment before starting the worker installation.

Next, type 
```bash
sudo ./install
```

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

!!! news
    The tutorial of this part : <a href="https://www.youtube.com/watch?v=Z_vaBbaMNkw" target="_blank">RemoteLabz Installation Part4</a>

You have to configure the worker IP on the web interface of the front by clicking on the button + and type its IP. If you use only one server for the front and the worker, you can put 127.0.0.1 .

![Add a worker](/images/Administrator/AddWorker.png)

On your worker, in the file `/opt/remotelabz-worker/config/packages/messenger.yaml`, you had now the following lines 


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

!!! warning
    Don't forget to modify your `/opt/remotelabz-worker/.env.local` file. You have to define the following parameters :
    ADM_INTERFACE; FRONT_SERVER_IP; and SSH_USER_PASSWD; SSH_USER_PRIVATEKEY_FILE; SSH_USER_PUBLICKEY_FILE if you have many workers

#### Copy certificate files from the front to the worker
Copy the two files `~/EasyRSA/RemoteLabz-WebServer.crt` and `~/EasyRSA/RemoteLabz-WebServer.key` to your **worker** in directory `/opt/remotelabz-worker/config/certs`

```bash
cd ~/EasyRSA
source /opt/remotelabz/.env.local
scp ~/EasyRSA/RemoteLabz-WebServer.crt user@${WORKER_SERVER}:~
sudo scp ~/EasyRSA/RemoteLabz-WebServer.key user@${WORKER_SERVER}:~
```

On the **worker**
```bash
cd ~
sudo mv RemoteLabz-WebServer.* /opt/remotelabz-worker/config/certs/
sudo sed -i "s/REMOTELABZ_PROXY_USE_WSS=0/REMOTELABZ_PROXY_USE_WSS=1/g" /opt/remotelabz-worker/.env.local
sudo service remotelabz-worker restart
```

!!! warning
    You need to use the same certificate between your front and the worker. Don't forget to copy them and to change it automatically if your certificate expired.


#### Start your RemoteLabz Worker service
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

If you have an error 500, do the following :
```bash
cd /opt/remotelabz
sudo chown -R www-data:www-data config/jwt
sudo chown -R www-data:www-data var
```

## Configure your RemoteLabz

    ### Check permission on log files

On the Front :
```bash
sudo chown -R remotelabz:www-data /opt/remotelabz/var
sudo chmod ug+w -R /opt/remotelabz/var
```

On the Worker :
```bash
sudo chown -R remotelabz-worker:www-data /opt/remotelabz-worker/var
sudo chmod ug+w -R /opt/remotelabz-worker/var
```

### Add a DHCP Service for your laboratory
In the device list, you will find a device with the name "Migration". This container will be used to configure, via the Sandbox function, a new container, called "Service" to provide a DHCP service to your laboratory. Each laboratory has its own DHCP service and its own network, so the RemoteLabz needs to configure this generic container to offer IP on the right network. For each lab, if you add the DHCP service container, it will be configured with the IP : IP_Gateway - 1. 
For example, if your attributed network is 10.10.10.0/27, your gateway will be 10.10.10.30 and you DHCP service container will have the IP 10.10.10.29 .

First : go to the sandbox menu and start the "Migration" device. Next, in the console of the started device, configure the network of the device (show the log, with "Show logs" button to know it) 

!!! tips
    Add an IP address `ip addr add X.X.X.X/M dev eth0`

    Add the default route `ip route add default via X.X.X.X`


Next, open the following file:
`vi /etc/systemd/resolved.conf`. In this file, uncomment the DNS= line by removing the # at the beginning, and set it to:
`DNS=1.1.1.1 9.9.9.9`

These are example DNS servers; you can replace them with your preferred DNS servers.

Next, type the following command :
```bash
systemctl restart systemd-resolved
sudo update-locale LANG=en_US.UTF-8
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

The line (`systemctl disable systemd-networkd`) is mandatory otherwise your container will not have any IP.

Your "Service" device, which is a container, is now ready.

First, stop the "Migration" device. Then, click "Export" and enter "Service" as the New Name. Finally, click the "Export Device" button.

You now have a new operating system and a new device called "Service". If you decide to add this device to your lab, it will act as a DHCP server.

The DHCP server is already configured to automatically assign IP addresses to your devices, using the appropriate network settings to provide them with Internet access.


!!! warning
    In the device menu, remove the "login" option from the control protocols, as users should not edit this VM.

!!! tips
    You can rename this "Service" device to "DHCP for internet"


![DHCP Service](/images/Administrator/DHCP-service.png)

The installation is finished and RemoteLabz application must be working now. In order to be fully usable, you'll have to change the parameter in the `/opt/remotelabz/.env.local` according to the following :

```bash
APP_MAINTENANCE=0
```
If you leave this value to 1, nobody, except the administrator will be able to use the application.

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