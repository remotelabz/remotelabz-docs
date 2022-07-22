# Configure RemoteLabz to use SSL

This section guides you through the configuration of SSL between all service of the RemoteLabz.

## Requirement
Your Remotelabz must work fine before to configure the SSL

- You must connect to a device of type QEMU
- You must connect to a device of type LXC

## Configure your Apache 2 with HTTPS (required if you want to use Shibboleth)

During the installation process, the file `200-remotelabz-ssl.conf` is copy in your `/etc/apache2/sites-available` directory. You have to modify the following lines to insert the right certificate files :
```bash
        SSLCertificateFile	/etc/apache2/RemoteLabz-WebServer.crt
        #SSLCertificateChainFile /etc/ssl/certs/remotelabz._INTERMEDIATE.cer
        SSLCertificateKeyFile /etc/apache2/RemoteLabz-WebServer.key
```
Two case, either you have an official certificate or you have to generate your own certificate.
### Official certificate

If you have an official certificate, you have to copy it in your `/etc/apache2` directory and rename it to `RemoteLabz-WebServer.crt` and `RemoteLabz-WebServer.key`

### Self-signed certificate
Execute the script 
```bash
cd ~
sudo remotelabz/bin/install_ssl.sh
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

!!! tips
    You can verify your certificate with the following command : 
    ```openssl x509 -noout -text -in /etc/apache2/RemoteLabz-WebServer.crt```

### Copy certificate files to the worker
Copy the two files `~/EasyRSA/RemoteLabz-WebServer.crt` and `~/EasyRSA/RemoteLabz-WebServer.key` to your **worker** in directory `/opt/remotelabz-worker/config/certs`

```bash
cd ~/EasyRSA
source /opt/remotelabz/.env.local
scp ~/EasyRSA/RemoteLabz-WebServer.crt user@${WORKER_SERVER}:~
scp ~/EasyRSA/RemoteLabz-WebServer.key user@${WORKER_SERVER}:~
```

On the **worker**
```bash
cd ~
sudo mv RemoteLabz-WebServer.* /opt/remotelabz-worker/config/certs/
sudo sed -i "s/REMOTELABZ_PROXY_USE_WSS=0/REMOTELABZ_PROXY_USE_WSS=1/g" /opt/remotelabz-worker/.env.local

```

!!! warning
    You need to use the same certificate between your front and the worker. Don't forget to copy them and to change it automatically if your certificate expired.

### Secure your Apache configuration
Add these lines in file `/etc/apache2/apache2.conf`
```bash
<FilesMatch "^\.git">
        Require all denied
</FilesMatch>
```
Do not forget to restart Apache service `sudo service apache2 restart`

### Secure your server from web intrusion
To avoid the scan url, you can use fail2ban to ban IP they scan the ssh or web access.

```bash
sudo apt-get update
sudo apt-get install -y fail2ban
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