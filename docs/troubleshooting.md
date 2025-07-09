# Troubleshooting

## 500 Internal Server Error on Login page
Wrong permission for config/jwt/
```bash
# Change owner of config/jwt/*
chown -R www-data:www-data config/jwt
```

## 500 Internal Server Error on Labs page
Wrong permission in var/cache/prod/
```bash
# Change owner of cache/prod/
chown -R www-data:www-data *
```

## Error in prod when update
If an error occurred when you update your version, from prod or older version, you can :

 - delete the `*.lock` file
 - check with `git status` if you have additional file on your filesystem and delete them
 - delete the `var/cache` directory

## No device templates are displayed on editor
If you want to add a device in the editor and no templates are displayed in the dropdown, you have to change the permission of config/templates:
```bash
chown www-data:www-data config/templates
chmod 664 config/templates
```

## Clone Ubuntu VM and DHCP client doesn't work

When you clone an Ubuntu VM, it doesn’t use its MAC address to obtain an IP address, but rather a random machine ID value stored in the `/etc/machine-id` file. So, after cloning an Ubuntu VM, you will have two VMs with the same IP address but different MAC addresses. To solve this problem, you need to change the machine ID value
```bash
sudo rm /etc/machine-id
sudo systemd-machine-id-setup
sudo reboot
```

## Error with numpy when installing

If the setup fails while processing numpy-2.1.3.tar.gz with this error:
    
`error: Couldn't find a setup script in /tmp/easy_install-cw60uo3_/numpy-2.1.3.tar.gz
ERROR: (`
    
 Install numpy manually with `pip install numpy.` and re-run the installation script.
    
##Remotelabz fail to install due to unrecognizable characters error in yarn.lock

This is due to a bug in SHA512 key generation in the yarn.lock file.
To correct this, open the Yarn.lock file in opt/remotelabz/ and seek those lines

```bash
"@babel/plugin-syntax-dynamic-import@^7.0.0", "@babel/plugin-syntax-dynamic-import@^7.8.3":
  version "7.8.3"
  resolved "https://registry.npmjs.org/@babel/plugin-syntax-dynamic-import/-/plugin-syntax-dynamic-import-7.8.3.tgz"
  integrity sha512-5gdGbFon+PszYzqs83S3E5mpi7/y/8M9eC90MRTZfduQOYW76ig6SOSPNe41IG5LoP3FGBn2N0RjVDSQiS94kQ==
=======
"@babel/plugin-syntax-class-static-block@^7.14.5":
  version "7.14.5"
  resolved "https://registry.npmjs.org/@babel/plugin-syntax-class-static-block/-/plugin-syntax-class-static-block-7.14.5.tgz"
  integrity sha512-b+YyPmr6ldyNnM6sqYeMWE+bgJcJpO6yS4QD7ymxgH34GBPNDM/THBh8iunyvKIZztiwLH4CJZ0RxTk9emgpjw==
  dependencies:
    "@babel/helper-plugin-utils" "^7.14.5"
```
And change it by

```bash
"@babel/plugin-syntax-dynamic-import@^7.0.0", "@babel/plugin-syntax-dynamic-import@^7.8.3":
  version "7.8.3"
  resolved "https://registry.npmjs.org/@babel/plugin-syntax-dynamic-import/-/plugin-syntax-dynamic-import-7.8.3.tgz"
  integrity sha512-5gdGbFon+PszYzqs83S3E5mpi7/y/8M9eC90MRTZfduQOYW76ig6SOSPNe41IG5LoP3FGBn2N0RjVDSQiS94kQ=========
"@babel/plugin-syntax-class-static-block@^7.14.5":
  version "7.14.5"
  resolved "https://registry.npmjs.org/@babel/plugin-syntax-class-static-block/-/plugin-syntax-class-static-block-7.14.5.tgz"
  integrity sha512-b+YyPmr6ldyNnM6sqYeMWE+bgJcJpO6yS4QD7ymxgH34GBPNDM/THBh8iunyvKIZztiwLH4CJZ0RxTk9emgpjw==
  dependencies:
    "@babel/helper-plugin-utils" "^7.14.5"
```

##Remotelabz fail to install due to chmod error in /home/hug/remotelabz/lib/remotelabz/Installer.php 

This is due to the installer script trying to change ownership and right to a nonexistent directory.
Open the file in a text editor and comment these lines should resolve the issue.
```bash       
  $this->rchown($this->installPath."config/templates", "www-data", "www-data");      
```

 






    


