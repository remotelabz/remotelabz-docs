# Profile layout

After login as an admin, you will be taken to your profile main page.

![Screenshot](/images/Administrator/Admin_Front.png)

The screen is divided in 4 main parts :

* The **navigation bar** with :
    * Lab Management : to create or stop labs.
    * Groups : all group that you belong to. You can also create new groups.
    * Bookings : to book a physical lab.
    * Admin area : to manage almost every aspects of the application.
    * Settings : to customise your profile.
* **Your labs** : displays labs that you can access from your groups.
* **Your bookings** : displays labs that you have booked.
* **Your groups** : displays groups that you belong to.


## Admin area

![Screenshot](/images/Administrator/User_indiv.png)

Inside this area, you can manage many elements such as :

* **Users** : can be added/edited/deleted from this area.
* **Flavors** : can be added/edited or deleted there. It corresponds to the device's memory presets which define which amount of memory a device can consume.
* **Control Protocols** : can be added/edited or deleted from there. It corresponds to the protocol used to access the device.
* **Operating Systems** : corresponds to the list of operating systems available to create a container.
* **Device Templates** : configuration templates for every available device can be added/edited or deleted from there.
* **Sandbox** : containers, vms can be started from there.New container can also be created from existing containers.
* **Instances** : allow the administrator to start/stop/leave every lab from there.
* **Services** : allow the administrator to start/stop critical system services such as : 
    * Worker service : the service that runs all the VM/Container for remotelabZ 
    * Proxy service : the service that control the private network used by the labs.
    * Messaging service : handles communications between the front and the worker. 
* **Usage** : give the administrator an overview about which resources are consumed by the worker.

## Profile restriction

The administrator can administrate all the application.
