# Hypervisor management
The hypervisor is the software that host the device.
![Screenshot](/images/Administrator/hypervisor/Administrator_Hypervisor.png)
By default, Remotelabz comes with three different hypervisors:

* **natif** : default hypervisor.
* **LXC** : host the device on lxc ( required for containers)
* **QEMU** : host the device on qemu (required for virtual machines).

## Hypervisor creation
By clicking on the `new hypervisor` button, you will be prompted to enter the new hypervisor name.
![Screenshot](/images/Administrator/hypervisor/Administrator_Hypervisor_add.png)

Here, you can enter the name of the new hypervisor and then click on `submit` to add it to the hypervisor's list.
!!!warning
    You can only add here the name of the hypervisor. Technical details of the hypervisor must be provided manually in RemoteLabz. This guide doesn't cover this topic yet.

## Hypervisor edition
Once selected, you can also edit the hypervisor's name through the `Edit` button or delete it through the `Delete` button.
!!!Warning
    Once you have clicked on the `Delete` button, the hypervisor's name will be definitely removed without any confirmation.
    
![Screenshot](/images/Administrator/hypervisor/Administrator_Hypervisor_selected.png)
Once you have pressed the `Edit` button,you will be able to edit the hypervisor through this screen. 
![Screenshot](/images/Administrator/hypervisor/Administrator_Hypervisor_edit.png)
