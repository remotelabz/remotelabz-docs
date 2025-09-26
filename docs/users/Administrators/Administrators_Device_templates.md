# Device Template management
From this area, you can manage every existing device templates.
![Screenshot](/images/Administrator/Device_templates/Administrator_device_template.png)
A device template is a model from which a lab device is created. You can create two types of device. It includes various parameters like:

* **Name** : name of the template.
* **Icon** : icon of the device.
* **Brand** : brand of the device.
* **Model** : model of the device.
* **Operating System** : corresponds to the device's Operating System.
* **Flavor** : memory profile it will use (control the amount of allocated RAM and disk).
* **Nb CPU** : number of CPU.
* **Nb Core** : number of core per CPU.
* **Nb Socket** : number of CPU Socket.
* **Nb Threads** : number of threads per CPU.
* **Control Protocol types** : control protocol used to administrate the device.
* **Template** : allow or disallow use in the lab editor.

Physical devices have 2 additional parameters :

* **IP** : IP address of the device.
* **Port** : port of the device.


# Device template creation
Once you have clicked on the `New Virtual Device` or `New Physical Device` button, you will be brought to a new screen. 
![Screenshot](/images/Administrator/Device_templates/Administrator_device_template_create.png)
As you finished adding your parameters, click on `Submit` to add your new device template.

When creating a virtual device, you can choose it to be a LXC container. It will complete some fields of the form and download the image. After clicking on the `New LXC container` button, you have to follow these steps :

* **Step 1** : Select the OS among 25 linux distribution.
* **Step 2** : Select a version for the OS.
* **Step 3** : Confirm your choice.

# Device template edition
Once selected, you can also edit the device's name through the `Edit Device` button or delete it through the `Delete Device` button.
![Screenshot](/images/Administrator/Device_templates/Administrator_device_template_select.png)
From there, you can consult informations like the type of the device, device informations, hardware specifications, control protocols, statistics and the icon of the device.

Once you have pressed the `Edit Device` button, you will be able to edit the Device through this screen
![Screenshot](/images/Administrator/Device_templates/Administrator_device_template_edit.png) 
Then click on the submit button to validate your changes and modify the selected device accordingly.
