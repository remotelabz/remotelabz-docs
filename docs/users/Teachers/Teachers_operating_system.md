# Operating System Management

!!! warning
    This page only concerns `Editor Teacher` role and not `Teacher`.


From this area, you can manage the operating system linked to a device template.
![Screenshot](/images/Administrator/Operating_system/Administrator_operating_system.png)
There is multiple default operating system provided with RemoteLabz.

!!! Tips
    The operating system can be hosted through a local image :material-upload: or a link where RemoteLabz can retrieve the image :material-link-variant:.  

## Operating System creation
### Operating System
To create an operating system, you have to click on the `New Operating System` button. You will be brought to a new screen.
![Screenshot](/images/Administrator/Operating_system/Administrator_new_operating_system.png)
From there, you can choose it's parameters such as :  

 * **Name** :  name of the new operating system.
 * **Architecture** : architecture under which processes will be executed.
 * **Hypervisor**:  hypervisor where the image will run. 
 * **Description** : optional description of the operating system.

According to the hypervisor type, other fields must be completed : 

 * **Template Name** : if the hypervisor type is LXC, a template name should be specified.
 * **Image Source Type**: if the hypervisor type is Qemu, it's the source of the image which can be an uploaded file (local file), an URL(link to a download server) or a filename.

!!! warning 
    You can only provide an image URL (link) or a file through the image file field, but not both !

### Blank OS
You can create a blank operating system by clicking on the `Blank OS` button.
![Screenshot](/images/Administrator/Operating_system/Administrator_blank_operating_system.png)

* **OS Name**: name of the operating system.
* **Disk Flavor**: optional disk size in gigabytes.
* **Description**: optional description of the operating system.

This blank OS will be created without any disk image, useful for testing purposes.

!!! warning 
    A blank OS should only be linked to one VM.

### LXC container
You can also create an LXC container, click on the `New LXC container`. You have to follow these steps :

* **Step 1** : Select an OS among 25 linux distribution. Images are downloaded from the official Linux Containers project repository.
![Screenshot](/images/Administrator/Operating_system/Administrator_operating_system_lxc1.png)
* **Step 2** : Choose a version for the OS.
![Screenshot](/images/Administrator/Operating_system/Administrator_operating_system_lxc2.png)
* **Step 3** : Confirm your choice.
![Screenshot](/images/Administrator/Operating_system/Administrator_operating_system_lxc3.png)

## Operating System edition
Once selected, you can also edit the Operating System's name through the `Edit Operating System` button or delete it through the `Delete Operating System` button.

![Screenshot](/images/Administrator/Operating_system/Administrator_operating_system_select.png)

After you have pressed the `Edit Operating System` button, you will be able to edit the Operating System through this screen. 

![Screenshot](/images/Administrator/Operating_system/Administrator_operating_system_edit1.png)

Click on the submit button to validate your changes.