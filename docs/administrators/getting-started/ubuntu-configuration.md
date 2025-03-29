
# Configure your RemoteLabz

## Add a DHCP Service for your laboratory
In the device list, you will find a device with the name "Migration". This container will be used to configure a new container, called "Service" to provide a DHCP service to your laboratory. Each laboratory has its own DHCP service and its own network, so the RemoteLabz needs to configure this generic container to offer IP on the right network. For each lab, if you add the DHCP service container, it will be configured with the IP : IP_Gateway - 1. 
For example, if your attributed network is 10.10.10.0/27, your gateway will be 10.10.10.30 and you DHCP service container will have the IP 10.10.10.29 .

First : go to the sandbox menu and start the "Migration" device. Next, in the console of the started device, configure the network of the device (show the log, with "Show logs" button to know it) 
!!! tips
    Add an IP address `ip addr add X.X.X.X/M dev eth0`

    Add the default route `ip route add default via X.X.X.X`


Next, type the following command :
```bash
sudo rm /etc/resolv.conf
echo "nameserver 1.1.1.1" > /etc/resolv.conf
apt-get update; apt-get -y upgrade; apt-get install -y dnsmasq;
echo "dhcp-range=RANGE_TO_DEFINED" >> /etc/dnsmasq.conf
echo "dhcp-option=3,GW_TO_DEFINED" >> /etc/dnsmasq.conf
systemctl stop systemd-resolved
systemctl disable systemd-resolved
systemctl disable systemd-networkd
systemctl enable dnsmasq
systemctl start dnsmasq
```

The last line (`systemctl disable systemd-networkd`) is mandatory otherwise your container will not have any IP.


Configuration example for a container that is connected to the network 10.11.18.0 :

```bash
ip addr add 10.11.18.3 dev eth0
ip route add default via 10.11.18.254 dev eth0 onlink
sudo rm /etc/resolv.conf
echo "nameserver 1.1.1.1" > /etc/resolv.conf
apt-get update; apt-get -y upgrade; apt-get install -y dnsmasq;
echo "dhcp-range=10.11.18.1,10.11.18.255" >> /etc/dnsmasq.conf
echo "dhcp-option=3,10.11.18.254" >> /etc/dnsmasq.conf
systemctl stop systemd-resolved
systemctl disable systemd-resolved
systemctl disable systemd-networkd
systemctl enable dnsmasq
systemctl start dnsmasq
```

Your "Service" device, which is a container, is now ready. You have to stop the Migration device, click on Export and type, as a New Name : Service and click on the button "Export Device"
On your lab, if you add Service device, you will have a DHCP service for all your devices of your lab.

## Configure RemoteLabz to use SSL

This section guides you through the configuration of SSL between all service of the RemoteLabz.

### Requirement
Prior to configure SSL, Remotelabz front and worker must be installed and fully functional. 

- You must connect to a device of type QEMU
- You must connect to a device of type LXC

### Configure your Apache 2 with HTTPS (required if you want to use Shibboleth)

During the installation process, the file `200-remotelabz-ssl.conf` is copied in your `/etc/apache2/sites-available` directory. The certificate is defined as follow :
```bash
        SSLCertificateFile	/etc/apache2/RemoteLabz-WebServer.crt
        #SSLCertificateChainFile /etc/ssl/certs/remotelabz._INTERMEDIATE.cer
        SSLCertificateKeyFile /etc/apache2/RemoteLabz-WebServer.key
```

Two case, either you have an official certificate or you have to generate your own certificate.
#### Official certificate

If you have an official certificate, you have to copy it in your `/etc/apache2` directory and rename it to `RemoteLabz-WebServer.crt` and `RemoteLabz-WebServer.key`. Next, you have to activate this site:
```bash
sudo a2ensite 200-remotelabz-ssl.conf
sudo a2enmod ssl
sudo service apache2 reload
```

#### Self-signed certificate
Execute the script 
```bash
cd ~
sudo remotelabz/bin/install_ssl.sh
```

### Redirection to https
Verify if your application is available with HTTPS and if it works fine, you can modify the `/etc/apache2/sites-available/100-remotelabz.conf` to redirect all HTTP request to HTTPS. 
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
Then restart apache
```bash
sudo systemctl restart apache2
```
Now, if you go to the your application's url with http, you should be redirected to HTTPS.

!!! tips
    You can verify your certificate with the following command : 
    ```bash
    openssl x509 -noout -text -in /etc/apache2/RemoteLabz-WebServer.crt
    ```

!!! warning 
    Don't forget to reload the Apache 2 service
    ```bash
    sudo service apache2 reload
    ```

#### Copy certificate files to the worker
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
