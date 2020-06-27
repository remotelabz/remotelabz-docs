# RemoteLabz's Worker installation guide

This section guides you through the installation of RemoteLabz's worker and its components on an Ubuntu system.

!!! note "Foreword"

    - This section has only been tested with **Ubuntu 20.04**
    - The first steps explains how to install [requirements](#requirements). You may skip these steps if those software are already present on your system and go to [Install RemoteLabz](#install-remotelabz).

## Retrieve the RemoteLabz source
A remotelabz directory will be create on your home directory.
```bash
cd ~
git clone https://gitlab.remotelabz.com/crestic/remotelabz-worker.git
```

## Requirements
### PHP

RemoteLabz requires PHP >= 7.3. You can install it manually or via `ppa:ondrej/php` repo, which builds PHP and uploads it to a repo for many Unix systems.

#### On Ubuntu 20.04 LTS
``` bash
sudo apt-get update
sudo apt install -y curl gnupg php zip unzip php-bcmath php-curl php-gd php-intl php-mbstring php-mysql php-xml php-zip ntp
```

### Composer

You may download Composer by following [official documentation](https://getcomposer.org/download/), but RemoteLabz is delivered with a copy of Composer that you can copy in a `bin` folder.
``` bash
sudo cp ~/remotelabz/composer.phar /usr/local/bin/composer
sudo chmod 755 /usr/local/bin/composer
```

### Node.js
The NodeJS's version in Ubuntu 20.04 LTS is too old. We recommand to use version 12 of NodeJS
``` bash
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### Yarn
``` bash
sudo npm install -g yarn
```

## Install RemoteLabz's worker

While you're in RemoteLabz root directory :

``` bash
sudo bin/install
```

Then, you should modify the `.env` file according to your environment, including SQL database variables with `MYSQL_SERVER`, `MYSQL_USER`, `MYSQL_PASSWORD` and `MYSQL_DATABASE`.

``` bash
sudo cp .env.dist .env
sudo nano .env
# you may change the MESSENGER_TRANSPORT_DSN variable with the following and with your credentials and server location
MESSENGER_TRANSPORT_DSN=amqp://remotelabz-amqp:password-amqp@localhost:5672/%2f/messages
```

### Instances

In order to be able to control instances on [the worker](https://gitlab.remotelabz.com/crestic/remotelabz-worker), you need to start **Symfony Messenger** :

```bash
sudo systemctl start remotelabz
```

**Warning :** When consuming messages, a timestamp is used to determine which messages the messenger worker is able to consume. Therefore, each machines needs to be time-synchronized. We recommand you to use a service like `ntp` to keep your machines synchronized.

