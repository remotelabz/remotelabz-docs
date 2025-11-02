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







    


