# RemoteLabz's front update guide

###Save your .env file
All configuration variables are saved in the file `.env.local` or if you don't have this file, in the file `.env`
If you don't have a .env.local, save your .env to .env.local
```bash
sudo -u remotelabz cp .env .env.local
```
Check your server.conf of your openvpn. Some deprecated parameters can be replaced.

###Update the code
```bash
cd /opt/remotelabz
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
sudo yarn encore prod
sudo php bin/console doctrine:migrations:migrate
sudo php bin/console cache:clear
sudo chown remotelabz:www-data * -R
sudo chmod g+w /opt/remotelabz/var -R
sudo chmod g+r config/jwt/private.pem
sudo systemctl daemon-reload
sudo service remotelabz restart
```

##Migration from 2.4.1.1 to 2.4.1.3
In your device, you have a device with the name "Migration". This container will be used to configure a new container, called "Service" to provide a DHCP service to each lab you will build.

First : in the sandbox, start the "Migration" device. In the console, configure the network of the device (show the log to know it) and next, type the following command :
```bash
apt-get update; apt-get -y upgrade; apt-get install -y dnsmasq;
echo "dhcp-range=RANGE_TO_DEFINED" >> /etc/dnsmasq.conf;
echo "dhcp-option=3,GW_TO_DEFINED" >> /etc/dnsmasq.conf;
sync;
systemctl enable dnsmasq;
sudo cp /opt/remotelabz/bin/remotelabz.service /etc/systemd/system
sudo systemctl daemon-reload
```
Modify your /etc/sudoers file to add, at the end, the two following lines: 
```bash
Cmnd_Alias REMOTELABZ_CMDS = /bin/systemctl start remotelabz*, /bin/systemctl stop remotelabz*, /bin/systemctl status remotelabz*

%www-data ALL=(ALL) NOPASSWD: REMOTELABZ_CMDS
```
Delete the two lines in your 


##Migration from 2.4.0 to 2.4.1.1

To update from 2.4.0 to 2.4.1.1, on the worker, you have to :
```bash
sudo lxc-create -t download -n Migration -- -d debian -r bullseye -a amd64 --keyserver hkp://keyserver.ubuntu.com
```
Create a new device like in the following image :
![Screenshot](/images/Migration/Migration.jpg)
In the Device Sandbox menu, click on the button "Modify" of the device "Migration" and start it (with button play). On the worker, a new container is created and started with an uuid name. You can find the uuid under the device name "Migration", like in the following image.
![Screenshot](/images/Migration/Migration-Sandbox.jpg)
In this example, the uuid is`a18d566a-6023-43df-a16e-4367065c5ecf` and on the worker, with the command `lxc-ls -f`, we can see this container.
![Screenshot](/images/Migration/Migration-Console.jpg)

Install in this container a dhcp server. The following example used the previous uuid.
```bash
sudo lxc-attach -n "a18d566a-6023-43df-a16e-4367065c5ecf"
echo "nameserver 1.1.1.3" > /etc/resolv.conf
apt-get update
apt-get upgrade
apt-get install dnsmasq
echo "dhcp-range=RANGE_TO_DEFINED" >> /etc/dnsmasq.conf
```
After, you can stop the container from the RemoteLabz interface and you have to export this modified device with the name "Service"
![Screenshot](/images/Migration/Migration-Export.jpg)

A new operating system and device is created with the name "Service" but the name of the image of the operating system is wrong. It's "service_" following a date. You have to edit the Operating System called "Service" with the Edit button. 
![Screenshot](/images/Migration/Migration-OS-image.jpg)

Change the image filename "service_" to "Service"
![Screenshot](/images/Migration/Migration-OS-changed.jpg)

Now, you can create a new lab with this device to offer IPv4 addresses to all other device, with the DHCP server installed in this container. The IPv4 of the DHCP will be the (last IPv4 - 1). The last IP is always the gateway of your laboratory.

![Screenshot](/images/Migration/Migration-End.jpg)


##Migration from 2.3 to 2.4.0
To update from 2.3 to 2.4.0, you can add default image to all labs with the following command :
```bash
sudo find /opt/remotelabz/public/uploads/lab/banner/* -type d -exec cp /opt/remotelabz/public/build/images/logo/nopic.jpg {}/nopic.jpg \;
```
You also have to add all user in the default group if you want they can execute some basic labs


