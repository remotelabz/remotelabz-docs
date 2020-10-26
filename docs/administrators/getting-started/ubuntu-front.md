# RemoteLabz's front installation guide

This section guides you through the installation of RemoteLabz and its components on an Ubuntu system. We assume you have already installed an Ubuntu Server 20.04 LTS.

!!! note "Foreword"

    - This section has only been tested with **Ubuntu 18.04 (LTS)** and **Ubuntu 20.04**.
    - The first steps explains how to install [requirements](#requirements). You may skip these steps if those software are already present on your system and go to [Install RemoteLabz](#install-remotelabz).

!!! warning
    - From the version 2.2.0 of RemoteLabz, we recommand to use at least Ubuntu 20.04 (LTS)

## Retrieve the RemoteLabz source
A remotelabz directory will be create on your home directory.
```bash
cd ~
git clone https://github.com/crestic-urca/remotelabz.git
```

You have now a directory `remotelabz` created on your home directory.

!!! warning
    If you want install only a specific version, you have to do the following instruction, for version 2.2.0 for example.
    ```bash    
    git clone https://github.com/crestic-urca/remotelabz.git --branch 2.2.0 --single-branch
    ```

## Requirements
### PHP

RemoteLabz requires PHP >= 7.3. You can install it manually or via `ppa:ondrej/php` repo, which builds PHP and uploads it to a repo for many Unix systems.
#### On Ubuntu 18.04 LTS
``` bash
sudo add-apt-repository ppa:ondrej/php
sudo apt-get update
sudo apt install -y curl gnupg php7.3 zip unzip php7.3-bcmath php7.3-curl php7.3-gd php7.3-intl php7.3-mbstring php7.3-mysql php7.3-xml php7.3-zip ntp
```

#### On Ubuntu 20.04 LTS
``` bash
sudo apt-get update
sudo apt install -y curl gnupg php zip unzip php-bcmath php-curl php-gd php-intl php-mbstring php-mysql php-xml php-zip ntp openvpn
```

### Composer

You may download Composer by following [official documentation](https://getcomposer.org/download/), but RemoteLabz is delivered with a copy of Composer that you can copy in a `bin` folder.
``` bash
sudo cp ~/remotelabz/composer.phar /usr/local/bin/composer
sudo chmod 755 /usr/local/bin/composer
```

### Node.js
The NodeJS's version in Ubuntu 18 and above is too old. We recommand to use version 12 of NodeJS
``` bash
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### Yarn
``` bash
sudo npm install -g yarn
```
### MySQL Server

#### On Ubuntu 18.04 LTS
Forthcoming

#### On Ubuntu 20.04 LTS
``` bash
sudo apt-get install -y mysql-server
sudo mysql_secure_installation
sudo mysql -u root -p
CREATE USER 'user'@'localhost' IDENTIFIED WITH mysql_native_password BY 'mysql-password';
CREATE DATABASE remotelabz;
GRANT ALL ON remotelabz.* TO 'user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
Choose a secure password to your MySQL server and you have to disable the remote access to your mysql server.

!!! info
    If you need to activate the remote access to your MySQL, you have to create a user like this :
    ``` bash
    CREATE USER 'user'@'%' IDENTIFIED WITH mysql_native_password BY 'mysql-password';
    GRANT ALL ON remotelabz.* TO 'user'@'%';
    FLUSH PRIVILEGES;
    ```
### RabbitMQ

To use RabbitMQ as messaging backend, you need the **php-amqp** extension :

#### On Ubuntu 18.04 LTS
```bash
sudo apt-get install -y rabbitmq-server php7.3-amqp
```

#### On Ubuntu 20.04 LTS
```bash
sudo apt-get install -y rabbitmq-server php-amqp
```

#### Configuration of RabbitMQ
The [worker] needs to connect to the RabbitMQ. We have to create a specific user to the RemoteLabz. Change the password 'password-amqp' in the following command
```bash
sudo rabbitmqctl add_user 'remotelabz-amqp' 'password-amqp'
sudo rabbitmqctl set_permissions -p '/' 'remotelabz-amqp' '.*' '.*' '.*'
```
Restart the RabbitMQ server
```bash
sudo service rabbitmq-server restart
```

### Configure OpenVPN

####Installation of Easy RSA 3.0
```bash
cd ~
# link to the latest version
wget https://github.com/OpenVPN/easy-rsa/releases/download/v3.0.8/EasyRSA-3.0.8.tgz
tar -xzf EasyRSA-3.0.8.tgz
cd EasyRSA-3.0.8
```

####Prepare configuration files
Create the `vars` file and add the following lines. You can change the value for your organisation
```bash
#File ~/EasyRSA-3.0.8/vars
set_var EASYRSA_BATCH           "yes"
set_var EASYRSA_REQ_CN         "RemoteLabz-VPNServer-CA"
set_var EASYRSA_REQ_COUNTRY    "FR"
set_var EASYRSA_REQ_PROVINCE   "Grand-Est"
set_var EASYRSA_REQ_CITY       "Reims"
set_var EASYRSA_REQ_ORG        "RemoteLabz"
set_var EASYRSA_REQ_EMAIL      "contact@remotelabz.com"
set_var EASYRSA_REQ_OU         "RemoteLabz-VPNServer"
set_var EASYRSA_ALGO           "ec"
set_var EASYRSA_DIGEST         "sha512"
set_var EASYRSA_CURVE          secp384r1
#5 ans de validité pour le CA
set_var EASYRSA_CA_EXPIRE      1825
#5 ans de validité pour les certificats
set_var EASYRSA_CERT_EXPIRE    1825
```

Edit the file `openssl-easyrsa.cnf`
```bash
#File ~/EasyRSA-3.0.8/openssl-easyrsa.cnf
nano ~/EasyRSA-3.0.8/openssl-easyrsa.cnf
```
and comment the line beginning with `RANDFILE`
```bash
#RANDFILE               = $ENV::EASYRSA_PKI/.rnd
```

####Generate the CA of your VPN server
```bash
./easyrsa init-pki
./easyrsa build-ca
```
Type a passphrase to secure the CA Key. For example, you can choose passphrase `R3mot3!abz-0penVPN-CA2020`

####Edit you .env or .env.local file
You have to add your passphrase in your `.env` RemoteLabz application
```bash
SSL_CA_KEY_PASSPHRASE="R3mot3!abz-0penVPN-CA2020"
```


####Build the certificate for the VPN server
Change the value of the Common Name (CN) in the vars file to now create the certificate file for your OpenVPN server
```bash
#File ~/EasyRSA-3.0.8/vars
set_var EASYRSA_REQ_CN         "RemoteLabz-VPNServer"
```

!!! warning
    If you do not change the CN of your VPN server, you will have an error message on the client because you have generated a self-signed certificate.

Now, we can generate the certificate of your VPN Server
```bash
./easyrsa gen-req RemoteLabz-VPNServer nopass
```

Sign the CA request certificate :
```bash
./easyrsa sign-req server RemoteLabz-VPNServer
```
and you have to type the choosen passphrase of your CA (`R3mot3!abz-0penVPN-CA2020`)

Copy of the previous generated keys in OpenVPN server directory (`/etc/openvpn/server`)
```bash
sudo cp pki/issued/RemoteLabz-VPNServer.crt /etc/openvpn/server
sudo cp pki/private/RemoteLabz-VPNServer.key /etc/openvpn/server
sudo cp pki/ca.crt /etc/openvpn/server
sudo cp pki/private/ca.key /etc/openvpn/server
```

####Configure a pre-shared key to sign the data
```bash
sudo openvpn --genkey --secret ta.key
sudo cp ta.key /etc/openvpn/server
```

####Configure Diffie-Hellman
```bash
cd /etc/openvpn/server
sudo openssl dhparam -out dh2048.pem 2048
```

####Affect the right permission to your certificate and key files
The application needs to access to the certificate and key files to generate the OpenVPN file for the clients.
```bash
sudo chgrp remotelabz /etc/openvpn/server -R
sudo chmod g+rx /etc/openvpn/server -R
```

####Configure OpenVPN server
Edit the `/etc/openvpn/server/server.conf` file to obtain the same than the following
```bash
port 1194
proto udp
dev tun
ca ca.crt
cert RemoteLabz-VPNServer.crt
key RemoteLabz-VPNServer.key
dh dh2048.pem
cipher AES-256-GCM
tls-auth ta.key 0
server 10.8.0.0 255.255.255.0
keepalive 10 120
persist-key
persist-tun
status /var/log/openvpn/openvpn-status.log
log /var/log/openvpn/openvpn.log
comp-lzo
verb 1
mute 20
explicit-exit-notify 1
push "route 10.10.10.0 255.255.255.0"
```

####Enable OpenVPN service on boot
`sudo systemctl enable openvpn-server@server`

####Start OpenVPN service
`sudo service openvpn-server@server start`

####Activate the forward between the interface
In the file `/etc/sysctl.conf`, looking for the line `#net.ipv4.ip_forward=1` and uncomment it. Then, reload this sysctl file
```bash
sudo sysctl --system
```

## Install RemoteLabz

While you're in RemoteLabz root directory :

``` bash
cd ~/remotelabz
sudo bin/install
```

Then, you should create the `.env.local` file and put the correct environment variables from the `.env` according to your environment, including SQL database variables with `MYSQL_SERVER`, `MYSQL_USER`, `MYSQL_PASSWORD` and `MYSQL_DATABASE`.

``` bash
cd /opt/remotelabz
# To allow the web server to store the log
sudo chown -R www-data:www-data var
sudo cp .env .env.local
sudo nano .env.local
# Replace the MYSQL_USER, MYSQL_PASSWORD, and MYSQL_DATABASE values to the right value
MYSQL_USER=user
MYSQL_PASSWORD=mysql-password
MYSQL_DATABASE=remotelabz
# you may change the MESSENGER_TRANSPORT_DSN variable with the following and with your credentials and server location
MESSENGER_TRANSPORT_DSN=amqp://remotelabz-amqp:password-amqp@localhost:5672/%2f/messages
```

Run the `remotelabz-ctl` configuration utility to setup your database :

```bash
sudo remotelabz-ctl reconfigure database
```

!!! info
    With the loaded fixtures, default credentials are :

    - Username : `root@localhost`
    - Password : `admin`

    You may change those values by using the web interface.

#### Generate API keys

In order for the app to work correctly, you must create a key pair for JWT. You can find detailed configuration in [the LexikJWTAuthenticationBundle doc](https://github.com/lexik/LexikJWTAuthenticationBundle/blob/master/Resources/doc/index.md#generate-the-ssh-keys).

At the root of your RemoteLabz folder:

```bash
sudo mkdir -p config/jwt
sudo openssl genpkey -out config/jwt/private.pem -aes256 -algorithm rsa -pkeyopt rsa_keygen_bits:4096
sudo openssl pkey -in config/jwt/private.pem -out config/jwt/public.pem -pubout
sudo chown -R www-data:www-data config/jwt
```

Don't forget to edit your `.env.local` :

```bash
# Replace 'yourpassphrase' by your actual passphrase
echo "JWT_PASSPHRASE=yourpassphrase" | sudo tee -a .env.local
```

### Configure the route from the front to the worker VM's network
We assume you have configure now all variables in your .env.local which was modified after a copy of the .env
```bash
source /opt/remotelabz/.env.local
sudo ip route add $BASE_NETWORK/$BASE_NETWORK_NETMASK via $WORKER_SERVER
```

### Instances

In order to be able to control instances on [the worker](https://gitlab.remotelabz.com/crestic/remotelabz-worker), you need to start **Symfony Messenger** :

```bash
sudo systemctl start remotelabz
```

!!! warning
    When consuming messages, a timestamp is used to determine which messages the messenger worker is able to consume. Therefore, each machines needs to be time-synchronized. We recommand you to use a service like `ntp` to keep your machines synchronized.

You will also need to start the proxy service to display VNC console :

```bash
sudo npm install -g configurable-http-proxy
```

If we have a certificate on your web site (and it's recommanded), you have to define the certificat configurable-http-proxy must use
```bash
sudo nohup configurable-http-proxy --port 8000 --log-level info --ssl-key /etc/ssl/private/remotelabz-private_key.key --ssl-cert /etc/ssl/certs/remotelabz-ssl_certificate.cer
```
or if you don't use certificate
```bash
nohup configurable-http-proxy --port 8000 --log-level info &
```
## Secure your Apache configuration (recommanded)
Modify the following line in file `/etc/apache2/conf-enabled/security.conf`
```bash
ServerTokens Minimal
#ServerTokens OS
#ServerTokens Full

ServerSignature Off
#ServerSignature On
```
Do not forget to restart Apache service `sudo service apache2 restart`

## Use HTTPS instead of HTTP (Optional but required if you want to use Shibboleth)
During the installation process, the file `200-remotelabz-ssl.conf` is copy in your `/etc/apache2/sites-available` directory. You have to modify the following lines to insert the right certificate files :
```bash
        SSLCertificateFile	/etc/ssl/certs/remotelabz.crt
        SSLCertificateChainFile /etc/ssl/certs/remotelabz._INTERMEDIATE.cer
        SSLCertificateKeyFile	/etc/ssl/private/remotelabz.key
```
You have now to activate the virtual site and the SSL module
```bash
sudo a2enmod ssl
sudo a2ensite 200-remotelabz-ssl.conf
sudo service apache2 restart
```
Verify your application is now available with HTTPS and if it works fine, you can modify the `/etc/apache2/sites-available/100-remotelabz.conf` to redirect all HTTP request to HTTPS. 
Activate the rewrite module
```bash
sudo a2enmod rewrite
```

Uncomment the following lines in the file `/etc/apache2/sites-available/100-remotelabz.conf`:
```bash
#<IfModule mod_rewrite.c>
#    RewriteEngine On
#    RewriteCond %{HTTPS} !=on
#    RewriteRule ^/?(.*) https://%{SERVER_NAME}/$1 [R,L]
#</IfModule>
```
Now, if you go to the your application's url with http, you should be redirected to HTTTS.

## Shibboleth (optional)

!!!warning
    You have to activate HTTPS to use Shibboleth authentification method

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
 - [RENATER Shibboleth Service Provider (SP) Configuration Guide](https://services.renater.fr/federation/documentation/guides-installation/sp3/chap04){target=_blank}

You can find all the configuration guides on the following site :

- [On Ubuntu 18.04 LTS](https://www.switch.ch/aai/guides/sp/installation/?os=ubuntu18){target=_blank}
- [On Ubuntu 20.04 LTS](https://www.switch.ch/aai/guides/sp/installation/?os=ubuntu20){target=_blank}

To enable Shibboleth site-wide, you need to change the value of `ENABLE_SHIBBOLETH` environment variable :

```bash
# .env.local
ENABLE_SHIBBOLETH=1
```