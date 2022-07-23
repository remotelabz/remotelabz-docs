# RemoteLabz's installation guide

This section guides you through the installation of RemoteLabz and its components on an Ubuntu system. We assume you have already installed an Ubuntu Server 20.04 LTS. We support only this Ubuntu version.

## Installation of the requirements
The first step is to install Ubuntu Server 20.04 LTS https://releases.ubuntu.com/20.04.4/ubuntu-20.04.4-live-server-amd64.iso on

- only one computer if you want to use the Front and the Worker on the same server
- one 2 computers if you want to separate your Front and your Worker.

## Installation of the front

### Retrieve the RemoteLabz Front source
A remotelabz directory will be create on your home directory.
```bash
cd ~
git clone https://github.com/crestic-urca/remotelabz.git --branch master
```

You have now a directory `remotelabz` created on your home directory.

!!! warning
    If you want to install a specific version, you have to do the following instructions. For version 2.4.1 for example.
    ```bash    
    git clone https://github.com/crestic-urca/remotelabz.git --branch 2.4.1 --single-branch
    ```
    ou for development version
    ```bash    
    git clone https://github.com/crestic-urca/remotelabz.git --branch dev
    ```

### Install the requirements
```bash
cd remotelabz
sudo ./bin/install_requirement.sh
```

After this process, you have to understand the following information :

#### RabbitMQ and MySQL pre-configurations
The MySQL is configured with the root password : "RemoteLabz-2022\$", and a user "user" is created with password "Mysql-Pa33wrd\$". It is recommend to change it after your RemoteLabz works fine.

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

#### OpenVPN pre-configuration
The default passphrase used during the `install_requirement.sh` process is `R3mot3!abz-0penVPN-CA2020`. You can find this value in your `.env` file

```bash
SSL_CA_KEY_PASSPHRASE="R3mot3!abz-0penVPN-CA2020"
```
If you decide to change it, don't forget to change it in the `/opt/remotelabz/.env.local`.

!!! warning
    The last line `push "route 10.11.0.0 255.255.0.0"` in your `/etc/openvpn/server/server.conf` must be modified if you modifies, in your `.env.local` file, the parameters of the two next lines 
    ```BASE_NETWORK=10.11.0.0
    BASE_NETWORK_NETMASK=255.255.0.0```
    This network will be the network used for your laboratory. Your user must have a route on its workstation to join, via his VPN, his laboratory. Be careful, this network must have to be different of the user network at home.
`

### (Optionnal) Configure the mail (Exim4)
1. Configure the /etc/aliases to redirect all mail to root to an existing user of your OS
2. Check the aliases with the command `exim -brw root`
3. Edit the file `/etc/exim4/exim4.conf.template` and locate the part "Rewrite configuration" to have, for example, the following lines :
```bash
######################################################################
#                      REWRITE CONFIGURATION                         #
######################################################################

begin rewrite

user@* myemail@domain.com FfrsTtcb
root@* myemail@domain.com FfrsTtcb
```
4. Update your exim configuration with command `sudo update-exim4.conf`, following the command `sudo service exim4 restart`
5. Check all addresses are rewritten with the command `exim -brw root`

### Install RemoteLabz application

The install process will create the directory `/opt/remotelabz`.

While you're in RemoteLabz root directory :

``` bash
cd ~/remotelabz
sudo ./bin/install
```
The install process can take 5 minutes

!!! info
    During the installation, some actions is done on the directory permission :
    ```bash
    chgrp remotelabz /etc/openvpn/server -R
    chmod g+rx /etc/openvpn/server -R
    ```

#### Configure the RemoteLabz database
Run the `remotelabz-ctl` configuration utility to setup your database :

```bash
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
sudo chown -R www-data:www-data config/jwt
sudo chown -R www-data:www-data var
# Replace 'yourpassphrase' by your actual passphrase
echo "JWT_PASSPHRASE=\"JWTTok3n\"" | sudo tee -a .env.local
```

!!! warning
    Avoid special character in the JWT, otherwise you will have some errors

!!! tips
    In order for the app to work correctly, a key pair is created for JWT. You can find detailed configuration in [the LexikJWTAuthenticationBundle doc](https://github.com/lexik/LexikJWTAuthenticationBundle/blob/master/Resources/doc/index.md#generate-the-ssh-keys).

#### Start the RemoteLabz Front

In order to be able to control instances on [the worker](https://gitlab.remotelabz.com/crestic/remotelabz-worker), you need to start **Symfony Messenger** :

```bash
sudo systemctl enable remotelabz
sudo systemctl enable remotelabz-proxy
sudo systemctl start remotelabz
sudo systemctl start remotelabz-proxy
```

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

## Installation of the Worker
### Retrieve the RemoteLabz Worker source 
A remotelabz directory will be create on your home directory.
```bash
cd ~
git clone https://github.com/crestic-urca/remotelabz-worker.git --branch master
cd remotelabz-worker
```

!!! tips
    If you want to install only a specific version, you have to do the following instruction, for version 2.4.1 for example.
    ```bash    
    git clone https://github.com/crestic-urca/remotelabz-worker.git --branch 2.4.1 --single-branch
    ```
    ou
    ```bash    
    git clone https://github.com/crestic-urca/remotelabz-worker.git --branch dev
    ```

### Installation of the RemoteLabz worker application
You should modify the `~/remotelabz-worker/.env` file according to your environment before start the worker installation.

``` bash
# In all environments, the following files are loaded if they exist,
# the later taking precedence over the former:
#
#  * .env                contains default values for the environment variables needed by the app
#  * .env.local          uncommitted file with local overrides
#  * .env.$APP_ENV       committed environment-specific defaults
#  * .env.$APP_ENV.local uncommitted environment-specific overrides
#
# Real environment variables win over .env files.
#
# DO NOT DEFINE PRODUCTION SECRETS IN THIS FILE NOR IN ANY OTHER COMMITTED FILES.
#
# Run "composer dump-env prod" to compile .env files for production use (requires symfony/flex >=1.2).
# https://symfony.com/doc/current/best_practices/configuration.html#infrastructure-related-configuration

###> symfony/framework-bundle ###
APP_ENV=prod
APP_SECRET=89080404df40bd0ed77c9ef887165cc4

# ADM_INTERFACE will be used to administrate the worker
ADM_INTERFACE="ensX"

### For data connexion of all VM
# Define your data network interface
# Your worker server must have 2 network card to isolate administrative network and data network of all VM
# DATA_INTERFACE will be used to communicate on the data network
DATA_INTERFACE=$ADM_INTERFACE
#DATA_INTERFACE="ensY"

# To avoid exchange between device, IPTables is used to authorize communication only between the bridge interface of the laboratory and the Internet interface
# The internet access must be on the DATA_INTERFACE for the device but, sometimes, the VPN and Internet access are on the ADM_INTERFACE of the Worker.
INTERNET_INTERFACE_ACCESS=$DATA_INTERFACE

DATA_NETWORK=10.22.128.0/24
DATA_INT_IP_ADDRESS=10.22.128.2/24
DATA_INT_GW=10.22.128.254

LAB_NETWORK=172.16.16.0/24
USE_SUDO_FOR_SYSTEM_COMMANDS=1

###> symfony/messenger ###
# Choose one of the transports below
# MESSENGER_TRANSPORT_DSN=amqp://guest:guest@localhost:5672/%2f/messages
MESSENGER_TRANSPORT_DSN=amqp://remotelabz-amqp:password-amqp@127.0.0.1:5672/%2f/messages
# MESSENGER_TRANSPORT_DSN=redis://localhost:6379/messages
###< symfony/messenger ###

# Network configuration
# Must be equal to the parameter of the variable "server" in the file /etc/openvpn/server/server.conf
# server 10.8.0.0 255.255.255.0
VPN_NETWORK=10.8.0.0
VPN_NETWORK_NETMASK=255.255.255.0

# If the VPN is on the worker then the parameter VPN_CONCENTRATOR_IP must be equal to "localhost" else it must be equal to the IP
# VPN_CONCENTRATOR_IP is used to define the route from the worker and laboratories to the VPN users
# VPN_CONCENTRATOR_INTERFACE defines the interface which is used to join the VPN users from the worker

VPN_CONCENTRATOR_IP="localhost"
VPN_CONCENTRATOR_INTERFACE="localhost"
# VPN_CONCENTRATOR_IP="10.22.128.1"
# VPN_CONCENTRATOR_INTERFACE=$DATA_INTERFACE

# Use secured websocket between client and VM
REMOTELABZ_PROXY_USE_WSS=0
REMOTELABZ_PROXY_SSL_KEY="/opt/remotelabz-worker/config/certs/RemoteLabz-WebServer.key"
#If intermediate certificate exist, you have to paste the cert and the intermediaire in the same .pem file
REMOTELABZ_PROXY_SSL_CERT="/opt/remotelabz-worker/config/certs/RemoteLabz-WebServer.crt"
```

Next, type 
```bash
sudo ./install
```
### Configuration of the worker

#### Start your RemoteLabz Worker service
```bash
sudo systemctl enable remotelabz-worker
sudo systemctl start remotelabz-worker
```

You can check the log of the worker in `/opt/remotelabz-worker/var/log/prod.log`

!!! warning
    When consuming messages, a timestamp is used to determine which messages the messenger worker is able to consume. Therefore, each machines needs to be time-synchronized. We recommend you to use a service like `ntp` to keep your machines synchronized.

The installation is finish and RemoteLabz application must be works. You have now to change the parameter in the `/opt/remotelabz/.env.local` to have the following

```bash
APP_MAINTENANCE=0
```
If you let the value 1, nobody can use the application.

If you have an error 500, do the following :
```bash
cd /opt/remotelabz
sudo chown -R www-data:www-data config/jwt
sudo chown -R www-data:www-data var
```

### Configure the DHCP Service container
In your device, you have a device with the name "Migration". This container will be used to configure a new container, called "Service" to provide a DHCP service to each lab you will build.

First : in the sandbox, start the "Migration" device. In the console, configure the network of the device (show the log to know it) and next, type the following command :
```bash
apt-get update; apt-get -y upgrade; apt-get install -y dnsmasq;
echo "dhcp-range=RANGE_TO_DEFINED" >> /etc/dnsmasq.conf
echo "dhcp-option=3,GW_TO_DEFINED" >> /etc/dnsmasq.conf
systemctl enable dnsmasq
```

Your "Service" container is now ready. You have to stop the Migration device, click on Export and type, as a New Name : Service and click on the button "Export Device"
On your lab, if you add Service device, you will have a DHCP service for all your devices of your lab.

## Other tasks for production environment

### Configure your logrotate on worker
On the front, this task is done during installation process.

Add the following file `/etc/logrotate.d/remotelabz-worker`
```bash
/opt/remotelabz-worker/var/log/*.log {
        daily
        missingok
        rotate 52
        size 100M
        compress
        notifempty
        create 664 remotelabz-worker www-data
        su remotelabz-worker www-data
}
```

### Secure the communication
If you want to secure all communication between the client, the Remotelabz front and the Remotelabz Worker, you have to follow the instruction of [page SSL](ubuntu-secure.md) 
