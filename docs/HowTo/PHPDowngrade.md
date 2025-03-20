# PHP Downgrade 

On ubuntu distribution above 20.04, php 8 or above is installed when running the install script.
To use RemotelabZ we have to downgrade this version to PHP 7.4.

## Front Side

!!! warning
    This must be done after running /bin/install_requirement.sh

First, you need to know which php version you use by entering this command.
```bash
php --version
```

on this example, we are on php 8.1
```bash
PHP 8.1.2-1ubuntu2.19 (cli) (built: Sep 30 2024 16:25:25) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.1.2, Copyright (c) Zend Technologies
    with Zend OPcache v8.1.2-1ubuntu2.19, Copyright (c), by Zend Technologies
```

Then you must remove this version and all dependencies
```bash
apt purge php8.1-cli.
```

Once done we will install php 7.4 and all dependencies required for remotelabz Front to work.First we need to add the corresponding repository.
```bash
sudo add-apt-repository ppa:ondrej/php
sudo apt update
```
Then we have to reinstall PHP alongside it's required modules for RemotelabZ according to this version

```bash
apt install php7.4 php7.4-bcmath php7.4-curl php7.4-gd php7.4-intl php7.4-mbstring php7.4-mysql php7.4-xml php7.4-zip php7.4-amqp
```
Once done, we can proceed with the rest of the installation.


## Worker Side

!!! warning
    This must be done before running sudo ./install

On recent distributions, php 8 or above is installed by default.

To use RemotelabZ we have to downgrade this version to PHP 7.4.
First, you need to know which php version you use by entering this command.

```bash
php --version
```

on this example, we are on php 8.1

```bash
PHP 8.1.2-1ubuntu2.19 (cli) (built: Sep 30 2024 16:25:25) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.1.2, Copyright (c) Zend Technologies
    with Zend OPcache v8.1.2-1ubuntu2.19, Copyright (c), by Zend Technologies
```

Then you must remove this version and all dependencies
```bash
apt purge php8.1-cli.
```

Once done we will install php 7.4 and all dependencies required for remotelabz Front to work.First we need to add the corresponding repository.

```bash
sudo add-apt-repository ppa:ondrej/php
sudo apt update
```

Then we have to edit the install script on the remotelabz-worker directory

```bash
cd ~/remotelabz-worker/
nano install
```

Change these lines 

```bash
apt-get install -y ntp apache2 php zip unzip qemu qemu-kvm openvswitch-switch git python3 python3-pip python3-openvswitch php-xml php-curl php-amqp logrotate lxc screen build-essential cmake libjson-c-dev libwebsockets-dev curl exim4
```

To 

```bash
apt-get install -y ntp apache2 php7.4 zip unzip qemu qemu-kvm openvswitch-switch git python3 python3-pip python3-openvswitch php7.4-xml php7.4-curl php7.4-amqp logrotate lxc screen build-essential cmake libjson-c-dev libwebsockets-dev curl exim4
```
Once done, we can proceed with the rest of the installation.
    



    


