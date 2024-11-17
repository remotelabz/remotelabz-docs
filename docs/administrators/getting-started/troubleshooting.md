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

## Error with numpy when installing

If the setup fails while processing numpy-2.1.3.tar.gz with this error:
    
`error: Couldn't find a setup script in /tmp/easy_install-cw60uo3_/numpy-2.1.3.tar.gz
ERROR: (`
    
 Install numpy manually with `pip install numpy.` and re-run the installation script.
    
## lxc error while creating alpine 3.15 when installing

This is due to the script asking to install a version of Alpine that is no longer maintained.
To circumvent this, the install script must be modified.
Open the install script and change those lines

```bash
LXC=`lxc-ls -name "Alpine3.15"`;
if [ "${LXC}" == "" ] ; then
echo "Creation of a container Alpine 3.15"
DOWNLOAD_KEYSERVER="keyserver.ubuntu.com" lxc-create -t download -n Alpine3.15 -- -d alpine -r 3.15 -a amd64
echo "No default login, please use Sandbox to configure a new OS from this" >> "/var/lib/lxc/Alpine3.15/rootfs/etc/issue"
echo "nameserver 1.1.1.3" > "/var/lib/lxc/Alpine3.15/rootfs/etc/resolv.conf"
echo "OK ✔️"
fi;    
```
to

```bash
LXC=`lxc-ls -name "Alpine3.17"`;
if [ "${LXC}" == "" ] ; then
echo "Creation of a container Alpine 3.17"
DOWNLOAD_KEYSERVER="keyserver.ubuntu.com" lxc-create -t download -n Alpine3.17 -- -d alpine -r 3.17 -a amd64
echo "No default login, please use Sandbox to configure a new OS from this" >> "/var/lib/lxc/Alpine3.17/rootfs/etc/issue"
echo "nameserver 1.1.1.3" > "/var/lib/lxc/Alpine3.17/rootfs/etc/resolv.conf"
echo "OK ✔️"
fi;
```

This will force the installer to install a version of Alpine that is still maintained (3.17).
Re-run the install script after this.


## Lab Export running endlessly
Restart the messenger first.
If the log mention :
```bash 
[2024-11-06T15:46:34.781498+01:00] messenger.CRITICAL: Error thrown while handling message Remotelabz\Message\Message\InstanceActionMessage. Removing from transport after 3 retries. Error: "Handling "Remotelabz\Message\Message\InstanceActionMessage" failed: Undefined class constant 'ACTION_EXPORT'" {"message":{"Remotelabz\\Message\\Message\\InstanceActionMessage":[]},"class":"Remotelabz\\Message\\Message\\InstanceActionMessage","retryCount":3,"error":"Handling \"Remotelabz\\Message\\Message\\InstanceActionMessage\" failed: Undefined class constant 'ACTION_EXPORT'","exception":"[object] (Symfony\\Component\\Messenger\\Exception\\HandlerFailedException(code: 0): Handling \"Remotelabz\\Message\\Message\\InstanceActionMessage\" failed: Undefined class constant 'ACTION_EXPORT' at /opt/remotelabz-worker/vendor/symfony/messenger/Middleware/HandleMessageMiddleware.php:80)\n[previous exception] [object] (Error(code: 0): Undefined class constant 'ACTION_EXPORT' at /opt/remotelabz-worker/src/MessageHandler/InstanceActionMessageHandler.php:141)"} []
```

This issue is caused by a state message EXPORT that is unrecognized by the worker.To solve this modify the file `/opt/remotelabz-worker/vendor/remotelabz/remotelabz-message-bundle/Message` and add `const ACTION_EXPORT = "export";` at the beginning of the file, where all others actions constants are listed.

Then restart the worker.

 






    


