# Ubuntu

This section guides you through the installation of RemoteLabz and its components on an Ubuntu system.

!!! note "Foreword"

    - This section has only been tested with **Ubuntu 18.04 (LTS)** and **Ubuntu 20.04**.
    - The first steps explains how to install [requirements](#requirements). You may skip these steps if those software are already present on your system and go to [Install RemoteLabz](#install-remotelabz).

## Retrieve the RemoteLabz source
A remotelabz directory will be create on your home directory.
```bash
cd ~
git clone https://gitlab.remotelabz.com/crestic/remotelabz.git
```

## Requirements
### PHP

RemoteLabz requires PHP >= 7.3. You can install it manually or via `ppa:ondrej/php` repo, which builds PHP and uploads it to a repo for many Unix systems.
#### On Ubuntu 18 LTS
``` bash
sudo add-apt-repository ppa:ondrej/php
sudo apt-get update
sudo apt install -y curl gnupg php7.3 zip unzip php7.3-bcmath php7.3-curl php7.3-gd php7.3-intl php7.3-mbstring php7.3-mysql php7.3-xml php7.3-zip
```

#### On Ubuntu 20.04 LTS
``` bash
sudo apt-get update
sudo apt install -y curl gnupg php zip unzip php-bcmath php-curl php-gd php-intl php-mbstring php-mysql php-xml php-zip
```


### Composer

You may download Composer by following [official documentation](https://getcomposer.org/download/), but RemoteLabz is delivered with a copy of Composer that you can copy in a `bin` folder.
``` bash
sudo cp composer.phar /usr/local/bin/composer
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

#### Ubuntu 20.04 LTS
``` bash
sudo apt-get install mysql-server

```

## Install RemoteLabz

While you're in RemoteLabz root directory :

``` bash
sudo bin/install
```

Then, you should modify the `.env` file according to your environment, including SQL database variables with `MYSQL_SERVER`, `MYSQL_USER`, `MYSQL_PASSWORD` and `MYSQL_DATABASE`.

``` bash
sudo cp .env.dist .env
# Replace 'mysqlpassword' by your actual password
echo "MYSQL_PASSWORD=mysqlpassword" | sudo tee -a .env

# or edit ENV file directly
sudo nano .env
```

!!! tip

    If you are using mysql >= 8.0 don't forget to create user with password plugin :

    ``` mysql
    CREATE USER 'user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
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
mkdir -p config/jwt
openssl genpkey -out config/jwt/private.pem -aes256 -algorithm rsa -pkeyopt rsa_keygen_bits:4096
openssl pkey -in config/jwt/private.pem -out config/jwt/public.pem -pubout
chown -R www-data:www-data config/jwt
```

Don't forget to edit your `.env` :

```bash
# Replace 'yourpassphrase' by your actual passphrase
echo "JWT_PASSPHRASE=yourpassphrase" | sudo tee -a .env
```

### Instances

In order to be able to control instances on [the worker](https://gitlab.remotelabz.com/crestic/remotelabz-worker), you need to start **Symfony Messenger** :

```bash
sudo systemctl start remotelabz
```

**Warning :** When consuming messages, a timestamp is used to determine which messages the messenger worker is able to consume. Therefore, each machines needs to be time-synchronized. We recommand you to use a service like `ntp` to keep your machines synchronized.

You will also need to start the proxy service to display VNC console :

```bash
sudo npm install -g configurable-http-proxy
# then start it
configurable-http-proxy
```

### RabbitMQ

To use RabbitMQ as messaging backend, you need the **php-amqp** extension :

```bash
sudo apt-get install -y php7.3-amqp
```

Then, modify the `.env` file according to your RabbitMQ configuration :

```bash
# you may change this string for your credentials and server location
MESSENGER_TRANSPORT_DSN=amqp://guest:guest@localhost:5672/%2f/messages
```

Don't forget to restart the messenger service :

```bash
sudo systemctl restart remotelabz
```

### Shibboleth (optional)

Follow [this guide](https://www.switch.ch/aai/guides/sp/installation/?os=ubuntu#2) to install Shibboleth on 18.04.

To enable Shibboleth site-wide, you need to change the value of `ENABLE_SHIBBOLETH` environment variable :

```bash
# .env
ENABLE_SHIBBOLETH=1
```