# Laboratory Management

## Basic Concepts

A laboratory is an exercise associated to devices. All devices are configured to the exercise but can be reused for other activities.

## Instance Management

In RemoteLabz, once a student join a laboratory, they can manage the newly created instance and its associated devices through the laboratory management panel.

![Screenshot](/images/Students/Student_Lab.png)

From this panel, as a student, you are allowed to : 

 - :material-tray-arrow-down: `OpenVPN File` : Download the OpenVPN file associated to the lab. This can be useful to connect through protocol like SSH.
 - `See Lab` : See the topology of the lab.
 - `Leave Lab` : Leave the lab. All devices must be stopped, this action will destroy the instance and all devices that are associated.

       For each device, you can : 

    - `start/stop` : Start and Stop the device.
    - `> Show logs` : Look at startup and shutdown log of the device.
    - `Lab Console` : Have a direct console access to the device. The attributed network to your laboratory and the gateway will be indicated.

Below the instance, you can see the practical subject of the lab if there is one.

!!!Warning
     Once the leave lab button has been clicked, it destroys the lab instance, erasing **ALL MODIFICATIONS** on the lab's devices.

From the instances list tab, you can see all lab instances that you joined. 

![Screenshot](/images/Students/Student_Instances.png)
By clicking the `Details` button, you have an overview of your instance as follows :

![Screenshot](/images/Students/Student_Instances_Details.png)

You can manage your devices and access to information like the network attributed to the lab instance.

## Lab Topology

![Screenshot](/images/Students/Student_lab_topology.png)

You can also see the topology of the lab and access to the practical subject and lab details through this screen as well.

##  Remote Access

In order to access to your lab from your workstation (through SSH for instance), you need to connect through RemoteLabz VPN.

To do this, you need to:

1. download a client like <a href="https://openvpn.net/client/">VPNconnect</a> and then install it.
2. click on the OpenVPN file button. An `.ovpn` file will be downloaded.
3. double-click on the downloaded file and VPNConnect will install it automatically.
4. hit the connect button.

If the client display this, you are connected.

<img src="/images/Students/VPN_Connect.png" height=600px width=300px>




