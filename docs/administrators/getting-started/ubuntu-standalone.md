# RemoteLabz's installation guide

!!! warning
    This section is in case you want to install RemoteLabz on one 1 computer.

This section guides you through the installation of RemoteLabz and its components on an Ubuntu system. We assume you have already installed an Ubuntu Server 20.04 LTS. We support only this Ubuntu version.

## Installation of the requirements
The first step is to install Ubuntu Server 20.04 LTS https://releases.ubuntu.com/20.04.4/ubuntu-20.04.4-live-server-amd64.iso

### Retrieve the RemoteLabz source
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
sudo ./install_requirement.sh
```

After this process, you have to understand the following informations :

#### RabbitMQ and MySQL pre-configurations
The MySQL is configured with the root password : "RemoteLabz-2022$", and a user "user" is created with password "Mysql-Pa33wrd$". It is recommend to change it after your RemoteLabz works fine.

!!! Tips
    During the `install_requirement.sh` process, a `remotelabz-amqp` user is created in RabbitMQ with the password `password-amqp`. If you want to change the password of an existing user `remotelabz-amqp` of your RabbitMQ, you have to type the following command :
    ```
    sudo rabbitmqctl change_password 'remotelabz-amqp' 'new_password'
    ```
    For MySQL, to modify your root password
    ```
    sudo mysql -u root -h localhost
    ALTER USER IF EXISTS 'root'@'localhost' IDENTIFIED BY 'new_password';
    FLUSH PRIVILEGES;
    EXITS;
    ```
    and to modify the remotelabz user password 
    ```
    ALTER USER IF EXISTS 'user'@'localhost' IDENTIFIED BY 'new_password';
    FLUSH PRIVILEGES;
    EXITS;
    ```

#### Configure OpenVPN
The default passphrase used in the `install_requirement.sh` is `R3mot3!abz-0penVPN-CA2020`. You can find this value in your `.env` file

```bash
SSL_CA_KEY_PASSPHRASE="R3mot3!abz-0penVPN-CA2020"
```
If you decide to change it, don't forget to change it in the `.env`.

!!! warning
    The last line `push "route 10.11.0.0 255.255.0.0"` in your `/etc/openvpn/server/server.conf` must be modified if you modifies, in your `.env.local` file, the parameters of the two next lines 
    ```BASE_NETWORK=10.11.0.0
    BASE_NETWORK_NETMASK=255.255.0.0```
    This network will be the network used for your laboratory. Your user must have a route on its workstation to join, via his VPN, his laboratory. Be careful, this network must have to be different of the user network at home.
`

### Configure the mail (Exim4)
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

## Install RemoteLabz

The install process will create the directory `/opt/remotelabz`. You can create a link to your home with the command `sudo ln -s ~/remotelabz /opt/remotelabz`

While you're in RemoteLabz root directory :

``` bash
cd ~/remotelabz
sudo bin/install
```
The install process can take 5 minutes

## Affect the right permission to your certificate and key files for OpenVPN
The application needs to access to the certificate and key files to generate the OpenVPN file for the clients.
```bash
sudo chgrp remotelabz /etc/openvpn/server -R
sudo chmod g+rx /etc/openvpn/server -R
```
## Configure you RemoteLabz database
Run the `remotelabz-ctl` configuration utility to setup your database :

```bash
sudo remotelabz-ctl reconfigure database
```

!!! info
    The default credentials are :

    - Username : `root@localhost`
    - Password : `admin`

    You may change those values by using the web interface.

#### Generate API keys

In order for the app to work correctly, you must create a key pair for JWT. You can find detailed configuration in [the LexikJWTAuthenticationBundle doc](https://github.com/lexik/LexikJWTAuthenticationBundle/blob/master/Resources/doc/index.md#generate-the-ssh-keys).

At the root of your RemoteLabz folder:

```bash
cd /opt/remotelabz
sudo mkdir -p config/jwt
sudo openssl genpkey -out config/jwt/private.pem -aes256 -algorithm rsa -pkeyopt rsa_keygen_bits:4096
#Your can use as passphrase "JWTTok3n"
sudo openssl pkey -in config/jwt/private.pem -out config/jwt/public.pem -pubout
sudo chown -R www-data:www-data config/jwt
```

Don't forget to edit your `.env.local` :

```bash
# Replace 'yourpassphrase' by your actual passphrase
echo "JWT_PASSPHRASE=\"JWTTok3n\"" | sudo tee -a .env.local
```

!!! warning
    Avoid special character in the JWT, otherwise you will have some errors


### Instances

In order to be able to control instances on [the worker](https://gitlab.remotelabz.com/crestic/remotelabz-worker), you need to start **Symfony Messenger** :

```bash
sudo systemctl enable remotelabz
sudo systemctl start remotelabz
```

!!! warning
    When consuming messages, a timestamp is used to determine which messages the messenger worker is able to consume. Therefore, each machines needs to be time-synchronized. We recommand you to use a service like `ntp` to keep your machines synchronized.

!!! warning
    Now you have to install RemoteLabz Worker

# Retrieve the RemoteLabz Worker source
A remotelabz directory will be create on your home directory.
```bash
cd ~
git clone https://github.com/crestic-urca/remotelabz-worker.git --branch master
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
```
Next, type 
```bash
sudo ./install
```
# Configuration
### Configure WSS
If you have configured the front with HTTPS, you have to copy your certificate to the `/opt/remotelabz-worker/config/certs/` directory, regarding the two parameters in your .env.local
`REMOTELABZ_PROXY_SSL_KEY` and `REMOTELABZ_PROXY_SSL_CERT`

!!! warning
    You need to use the same certificate between your front and this worker. Don't forget to copy them and to change it automatically when your certificate expired.

### Instances
```bash
sudo systemctl enable remotelabz-worker
sudo systemctl start remotelabz-worker
```

You can check the log of the worker in `/opt/remotelabz-worker/var/log/prod.log`

!!! warning
    When consuming messages, a timestamp is used to determine which messages the messenger worker is able to consume. Therefore, each machines needs to be time-synchronized. We recommand you to use a service like `ntp` to keep your machines synchronized.

The installation is finish and RemoteLabz application must be works. You have now to change the parameter in the /opt/remotelabz/.env.local to have the following

```bash
APP_MAINTENANCE=0
```

If you let the value 1, nobody can use the application.


## Configure your logrotate
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

### Configure your container
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

## Secure your Apache configuration (recommended)
Modify the following line in file `/etc/apache2/conf-enabled/security.conf`
```bash
ServerTokens Prod
#ServerTokens OS
#ServerTokens Full

ServerSignature Off
#ServerSignature On
```

Add these lines in file `/etc/apache2/apache2.conf`
```bash
<FilesMatch "^\.git">
        Require all denied
</FilesMatch>
```

Do not forget to restart Apache service `sudo service apache2 restart`

## Secure your server from web intrusion (recommended)
To avoid the scan url, you can use fail2ban to ban IP they scan the ssh or web access.

### On Ubuntu 20.04 LTS Server

```bash
sudo apt-get update
sudo apt-get install -y fail2ban
sudo service fail2ban restart
```

!!! warning
    This configuration can blocked your access because some request response stay in 404 when the device is not started. The following configuration is not yet recommended.

    1. At the end of the file `/etc/fail2ban/jail.conf`, add the following
    ```bash
    [apache-404]
    enabled = true
    port = http,https
    logpath = /var/log/apache2/access*.log
    #If find 3 404 errors during the findtime
    maxretry = 3
    #Ban for 1h
    bantime = 3600
    #In 600 seconds
    findtime = 600
    ```
    2. Create the file `/etc/fail2ban/filter.d/apache-404.conf` with this following content
    ```bash
    [Definition]
    failregex = ^<HOST> - .* "(GET|POST|HEAD).*HTTP.*" 404 .*$
    ignoreregex =.*(robots.txt|favicon.ico|jpg|png)
    ```
    3. Restart the fail2ban service
    ```bash
    sudo service fail2ban restart
    ```

Verify your fail2ban is up
```bash
sudo service fail2ban status
```

You will find in the log file `/var/log/fail2ban.log` all the rules you applied, banned IPs or restored access to a previously banned IP.

If you want to reinforce the security of the access to your server, you can modify the default value of fail2ban. For example, in file `/etc/fail2ban/jail.conf`, you can uncomment the following line in the [DEFAULT] section.
```bash
bantime.increment = true
```
and change the default value of `bantime` and `findtime` to 1 hour instead of 10 minutes
```bash
bantime  = 1h
findtime  = 1h
```

## Use HTTPS instead of HTTP (Optional but required if you want to use Shibboleth)
During the installation process, the file `200-remotelabz-ssl.conf` is copy in your `/etc/apache2/sites-available` directory. You have to modify the following lines to insert the right certificate files :
```bash
        SSLCertificateFile	/etc/apache2/RemoteLabz-WebServer.crt
        #SSLCertificateChainFile /etc/ssl/certs/remotelabz._INTERMEDIATE.cer
        SSLCertificateKeyFile /etc/apache2/RemoteLabz-WebServer.key
```
### Official certificate

You have just to import your certificate on your the front and on the worker

### My own certificate

To generate a self-signed certificate for your webserver, you have to follow the next steps :

```bash
cd ~/EasyRSA
cp ~/remotelabz/config/apache/cert.cnf .
```
Edit the `~/remotelabz/config/apache/cert.cnf` file and modify the value of variable `commonName` and `IP.1` with your domain and IP of your web server, respectively.
```bash
#File ~/remotelabz/config/apache/cert.cnf
[req]
default_bits  = 2048
distinguished_name = req_distinguished_name
req_extensions = req_ext
x509_extensions = v3_req
prompt = no
[req_distinguished_name]
countryName = FR
stateOrProvinceName = Marne
localityName = Reims
organizationName = RemoteLabz
commonName = 192.168.11.131
#commonName = mydomain.com
[req_ext]
subjectAltName = @alt_names
[v3_req]
subjectAltName = @alt_names
[alt_names]
IP.1 = 192.168.11.131
```

You can now use `openssl` to generate your self-signed certificate.
```bash
cd ~/EasyRSA
openssl req -x509 -nodes -days 365 -sha512 -newkey rsa:2048 -keyout RemoteLabz-WebServer.key -out RemoteLabz-WebServer.crt -config ~/remotelabz/config/apache/cert.cnf
sudo cp /home/florent/EasyRSA/RemoteLabz-WebServer.crt /etc/apache2/
sudo cp /home/florent/EasyRSA/RemoteLabz-WebServer.key /etc/apache2/
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

You can verify your certificate with the following command : 
```openssl x509 -noout -text -in RemoteLabz-WebServer.crt```

If you use only https, you need to also use WSS for all websocket connection. So, in your .env.local, you have to change the value of `REMOTELABZ_PROXY_USE_WSS=1` and copy the two files 
`home/florent/EasyRSA/RemoteLabz-WebServer.crt` and `home/florent/EasyRSA/RemoteLabz-WebServer.key` on the worker. **On the worker**, you also have to modify the `.env.local`

### Copy certificate files to the worker
We assume your `.env.local` is well configured. Change the `user` login in the following command.
```
cd /opt/remotelabz
source .env.local
scp ~/EasyRSA/RemoteLabz-WebServer.crt user@${WORKER_SERVER}:~/
scp ~/EasyRSA/RemoteLabz-WebServer.key user@${WORKER_SERVER}:~/
```


## Shibboleth (optional)

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
 - [RENATER Shibboleth Service Provider (SP) Configuration Guide](https://services.renater.fr/federation/documentation/guides-installation/sp3/chap04){target=_blank}

You can find all the configuration guides on the following site :

- [On Ubuntu 20.04 LTS](https://www.switch.ch/aai/guides/sp/installation/?os=ubuntu20){target=_blank}

To enable Shibboleth site-wide, you need to change the value of `ENABLE_SHIBBOLETH` environment variable :

```bash
# .env.local
ENABLE_SHIBBOLETH=1
```