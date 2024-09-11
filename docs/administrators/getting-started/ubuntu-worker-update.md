# RemoteLabz's front update guide

###Save your .env file
All configuration variables are saved in the file `.env.local` or if you don't have this file, in the file `.env`
If you don't have a .env.local, save your .env to .env.local
```bash
sudo -u remotelabz cp .env .env.local
```

###Update the code
```bash
cd /opt/remotelabz-worker
git fetch
git pull
```

!!! info
    To choose a specific version :
    ```bash
    git checkout tags/2.X.Y
    ```
    or a branch
    ```bash
    git checkout dev
    ```

Compare your `.env.local` file with the `.env` file to be sure we have the same parameters defined 

```bash
sudo composer update
sudo php bin/console cache:clear
sudo chown remotelabz-worker:www-data * -R
sudo chmod g+w /opt/remotelabz-worker/var -R
sudo systemctl daemon-reload
sudo service remotelabz-worker restart
```

## From 2.4.3 and above to version 2.5.0
When you add a worker on the front, you have to add the following lines on the `messenger.yaml` file, in the part 
```bash
framework:
    messenger:
        transports:
            async: '%env(MESSENGER_TRANSPORT_DSN)%'
            worker: 
                dsn: '%env(MESSENGER_TRANSPORT_DSN)%'
                options:
```
On your first worker Worker_1

```bash
queues:
    messages_worker1:
        binding_keys: [Worker_1 IP]
```

On your second worker Worker_2
```bash
queues:
    messages_worker2:
        binding_keys: [Worker_2 IP]
```
And so on...


## From 2.4.1.2 and above to version 2.4.1.3

You have to install ttyd package

```bash
sudo apt-get install -y screen build-essential cmake git libjson-c-dev libwebsockets-dev
cd ~
git clone https://github.com/tsl0922/ttyd.git
cd ttyd && mkdir build && cd build
cmake ..
make && sudo make install
```

You have also to install some default container
```bash
sudo lxc-create -t download -n Migration -- -d debian -r bullseye -a amd64 --keyserver hkp://keyserver.ubuntu.com;
sudo lxc-create -t download -n Debian -- -d debian -r bullseye -a amd64 --keyserver hkp://keyserver.ubuntu.com;
sudo lxc-create -t download -n Ubuntu20LTS -- -d ubuntu -r focal -a amd64 --keyserver hkp://keyserver.ubuntu.com;
sudo lxc-create -t download -n Alpine3.15 -- -d alpine -r 3.15 -a amd64 --keyserver hkp://keyserver.ubuntu.com;
sudo su;
echo "nameserver 1.1.1.3" > "/var/lib/lxc/Migration/rootfs/etc/resolv.conf";
echo "nameserver 1.1.1.3" > "/var/lib/lxc/Debian/rootfs/etc/resolv.conf";
echo "No default login, please use Sandbox to configure a new OS from this" >> "/var/lib/lxc/Debian/rootfs/etc/issue";
echo "No default login, please use Sandbox to configure a new OS from this" >> "/var/lib/lxc/Ubuntu20LTS/rootfs/etc/issue";
echo "No default login, please use Sandbox to configure a new OS from this" >> "/var/lib/lxc/Alpine3.15/rootfs/etc/issue";
echo "nameserver 1.1.1.3" > "/var/lib/lxc/Alpine3.15/rootfs/etc/resolv.conf";
exit;
```

```bash
echo "%remotelabz-worker     ALL = (ALL) NOPASSWD: $(which ip), $(which iptables), $(which ovs-vsctl), $(which systemctl) start remotelabz*, $(which systemctl) stop remotelabz*, $(which systemctl) restart remotelabz*, $(which systemctl) status remotelabz*" | sudo tee /etc/sudoers.d/remotelabz-worker
echo "%www-data     ALL = (ALL) NOPASSWD: $(which ip), $(which iptables), $(which ovs-vsctl), $(which systemctl) start remotelabz*, $(which systemctl) stop remotelabz*, $(which systemctl) restart remotelabz*, $(which systemctl) status remotelabz*" | sudo tee -a /etc/sudoers.d/remotelabz-worker
echo "net.ipv6.route.max_size = 20000" | sudo tee -a /etc/sysctl.conf
```