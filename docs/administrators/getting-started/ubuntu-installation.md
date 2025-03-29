# RemoteLabz's installation guide

This section guides you through the installation of RemoteLabz and its components on an Ubuntu system. We assume you have already installed an Ubuntu Server 20.04 LTS.For now, we support only this version of Ubuntu.

## Requirements

Only Ubuntu-based distributions are compatible with RemotelabZ.


The first step is to install a ubuntu distro like Ubuntu Server 20.04 LTS https://releases.ubuntu.com/20.04.4/ubuntu-20.04.4-live-server-amd64.iso on

- only one computer if you want to use the Front and the Worker on the same server
- 2 computers if you want to separate your Front and your Worker.

To install both the Front and the Worker on the same device, the minimum requirement is 

- a hard disk of at least 30 Go.
- 2 Go of RAM
- 1 CPU

This depends of the number of VMs, containers, and, operating system used, you want to run simultaneously. At the end of the installation, 4 devices will be installed and configured :

- 3 containers with Debian 11.4, Alpine 3.15, Ubuntu Server 20.04 LTS
- 1 VM Alpine 3.10

The 5th device, called "Migration" is another Alpine used for configuration.At the end of the installation, a 6th container with a DHCP service must be created.

!!! warning
    RemoteLabZ require PHP 7.4 to work properly. PHP 8.0 or higher is not supported.To downgrade PHP see [PHP Downgrade](../../../HowTo/PHPDowngrade)

## Installation of the front

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
    or for development version
    ```bash    
    git clone https://github.com/remotelabz/remotelabz.git --branch dev
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
If you decide to change it, don't forget to do this in the `/opt/remotelabz/.env.local`.

!!! warning
    The last line `push "route 10.11.0.0 255.255.0.0"` in your `/etc/openvpn/server/server.conf` must be modified if you modify, in your `.env.local` file, the parameters of the two next lines 
    ```BASE_NETWORK=10.11.0.0
    BASE_NETWORK_NETMASK=255.255.0.0```
    This network will be the network used for your laboratory. Your user must have a route on its workstation to join, via his VPN, his laboratory. Be careful, this network must be different of your home network.
`

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

### Install RemoteLabz application

The install process will create the directory `/opt/remotelabz`.

While you're in RemoteLabz root directory :

``` bash
cd ~/remotelabz
sudo ./bin/install
```
The install process may take around 5 minutes

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
    In order for the app to work correctly, a key pair is created for JWT. You can find detailed configuration in [the LexikJWTAuthenticationBundle doc](https://github.com/lexik/LexikJWTAuthenticationBundle/blob/master/Resources/doc/index.rst#generate-the-ssh-keys).


#### Directory permissions

To work properly, RemoteLabZ need the right to add/delete/modify files on the following directories:

 - <code>/opt/remotelabz/var</code> this is where RemotelabZ store it's logs and cached data.To set proper permissions on this directory, you need to enter as administrator <code>chmod -R 775 /opt/remotelabz/var </code>
 - <code>/opt/remotelabz/config/templates</code> this is where RemotelabZ store it's container files.To set proper permissions on this directory, you need to enter as administrator <code>chmod -R 775 /opt/remotelabz/config/templates</code>


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
    or
    ```bash    
    git clone https://github.com/remotelabz/remotelabz-worker.git --branch dev
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
    

### Configuration of the worker
You have to configure, first, at least, 1 worker, from your front server.This is done by modifying `\opt\remotelabz\config\packages\messenger.yaml`  When you add a worker to the front, you will have to add the following lines 
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

If you add another worker, you will have to add,
```bash
    messages_worker2:
        binding_keys: [Worker_2_IP]
```
and so on.
!!! warning
    If the worker is on a distant server, you must check if the worker IP is correctly set in `/opt/remotelabz/.env.local` according to the following :
    Example with a worker on a server with the IP 123.12.167.22
    ```bash
    WORKER_SERVER=123.12.167.22
    ```

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

The installation is finished and RemoteLabz application must be working now.In order to be fully usable, you'll have to change the parameter in the `/opt/remotelabz/.env.local` according to the following :

```bash
APP_MAINTENANCE=0
```
If you leave this value to 1, nobody will be able to use the application.

If you have an error 500, do the following :
```bash
cd /opt/remotelabz
sudo chown -R www-data:www-data config/jwt
sudo chown -R www-data:www-data var
```

