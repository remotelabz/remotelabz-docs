# Commonly Asked Questions

## RemoteLabz Logs location 

RemoteLabz's logs are located under `/opt/remotelabz/var/log/` and RabbitMQ's log under `/var/log/rabbitmq`

## Change password of phpmyadmin
```bash
sudo mysql -h localhost -u root -p
CREATE USER 'phpmyadmin'@'localhost' IDENTIFIED BY '<New-Password-Here>';
GRANT ALL PRIVILEGES ON *.* TO 'phpmyadmin'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```
If you have finish with your modification, you have to go lab and delete the new lab created from the sandbox.

## List the RabbitMQ messages waiting
On the RabbitMQ server
To list the number of message in each queues
```bash
sudo rabbitmqctl list_queues
```

## How to modify a device's image
To avoid a teacher or an user include a corrupted image, only the administrator can modify an existing default image. With the menu `Device Sandbox`, a new lab is created and after the export button is click, a new device template and new Operating Systems are created. To make an export, you have to :

1. Click on `Device Sandbox`
2. Click on the Modify button of the device whose image you wants to modify
3. Start the device
4. Make your modification
5. Stopped the device
6. Click on Export and choose a name for you new device template
7. Leave the lab

## How to increase size of disk on LVM virtual machines
1. Shutdown the VM
2. Right click the VM and select Edit Settings
3. Select the hard disk you would like to extend
4. On the right side, make the provisioned size as large as you need it and confirm
5. Power on the VM and connect to it.
6. Identify your disk name with `sudo fdisk -l` for example /dev/sda
7. `sudo fdisk /dev/sda`
8. Enter `p` to print the partition table
9. Enter `n` to add a new partition
10. Enter `p` again to make it a primary partition
11. Enter the number of your new partition
12. Pick the first cylinder which will most like come at the end of the last partition (this is the default value)
13. Enter the amount of space (default is the rest of space available)
14. Enter `w` to save these changes
15. Restart the VM and log in
16. Type `sudo fdisk -l` and check that a new partition is present
17. Find your volume group with `df -h`.
    * Example: `/dev/mapper/ubuntu--vg-root 15G 4.5G ...`
    * Volume group is: `ubuntu-vg`
18. Extend the volume group : `sudo vgextend [volume group] /dev/sdaX`
    * Example: `sudo vgextend ubuntu-vg /dev/sda3`
19. Find the amount of free space available : `sudo vgdisplay [volume group] | grep "Free"`
20. Expand the logical volume : `sudo lvextend -L+[freespace]G /dev/[volgroup]/[volume]`
     * Example 1: `sudo lvextend -L+64G /dev/ubuntu-vg/root`
     * Example 2: `sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu`
21. Expand the ext3 file system in the logical volume : `sudo resize2fs /dev/[volgroup]/[volume]`
     * Example: `sudo resize2fs /dev/ubuntu-vg/root`
22. You can now run the df command to very that you have more space `df -h`

## Extend physical partition
If you have a diffenre between the size of your disk and the logical size of your part, perhaps, the partition does not use all your disk.

1. `sudo parted /dev/sda`
2. In parted, type the following command
    * `(parted) print`
    * `(parted) resizepart 3 100%`
    * `(parted) quit`
3. `sudo pvresize /dev/sda3`
4. `sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv`
5. `sudo resize2fs /dev/ubuntu-vg/ubuntu-lv`

## How to import a new image from an OVA
We assume your image has the name `import_vm.ova` in OVF format and in your home directory.
```bash
cd ~
mkdir new_image
cd new_image
tar xvf ../import_vm.ova 
qemu-img convert import_vm.vmdk new_image.img -O qcow2
sudo mv new_image.img /opt/remotelabz-worker/images/
```
As root, in operating system (OS), add a new operating system and in the Image filename parameter, give the name of your new image file. For our example, you have to type "new_image.img". Finally, create a new device which use this OS.

## How to remove all bridge OVS on a worker manually
```bash
for i in $(sudo ovs-vsctl show | grep "Bridge" | grep "br-" | cut -d " " -f 6); do sudo ovs-vsctl del-br $i; done;
```

## How to check the queue and exchange with RabbitMQ
To list the queues
```bash
rabbitmqadmin list queues
```

To list the exchange
```bash
rabbitmqadmin list exchanges
```

To delete an exchange
```bash
sudo rabbitmqadmin delete exchange name="worker"
```

To create an exchange with a specific type (direct or fanout)
```bash
sudo rabbitmqadmin declare exchange name="worker" type="direct"
```

To clear a queue
```bash
sudo rabbitmqadmin purge_queue queue_name
```

## Configure OpenVSwitch

Visualisation des configurations des ports
```bash
ovs-vsctl list interface eth1

# Voir seulement les options configurées
ovs-vsctl get interface eth1 options

# Voir la vitesse configurée
ovs-vsctl get interface eth1 options:link-speed
ovs-vsctl get interface eth1 options:duplex
```

Liste des bridges et de ses ports
```bash
ovs-vsctl list-br  # Liste des bridges
ovs-vsctl list-ports br0  # Ports d'un bridge spécifique
```
    
Configurer la vitesse d'un port et son duplex
```bash
ovs-vsctl set interface eth1 options:link-speed=100
ovs-vsctl set interface eth1 options:duplex=full
```
Ou en une seule commande :
```bash
ovs-vsctl set interface eth1 options:link-speed=100 options:duplex=full
```

Traffic Control
```bash
tc qdisc add dev eth1 root handle 1: tbf rate 100mbit burst 32kbit latency 400ms
```

Delete a vlan tag
```bash
ovs-vsctl remove port <port_name> tag X
```

Add a vlan tag
```bash
ovs-vsctl set port <port_name> tag <vlan>
```

Delete a trunk
```bash
ovs-vsctl set port <port_name> trunks=[]
```