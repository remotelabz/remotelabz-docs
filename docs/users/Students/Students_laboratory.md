# Laboratory management

## Basic concepts

A laboratory is an exercise associated with devices. All devices are configured for the exercise, but they can be reused for other activities.

## Instance management

In RemoteLabz, once a student joins a laboratory, they create an instance of this lab. They can manage this instance and its associated devices through the laboratory management panel.

![Screenshot](/images/Students/Student_Lab.png)

From this panel, as a student, you are allowed to:

- :material-tray-arrow-down: `OpenVPN File`: download the OpenVPN file associated with the lab. This is useful to connect through a protocol such as SSH.
- `See Lab`: see the topology of the lab.
- `Leave Lab`: leave the lab. All devices must be stopped. This action will destroy the instance and all the devices associated with it.

For each device, you can:

- `start/stop`: start and stop the device.
- `> Show logs`: look at the startup and shutdown log of the device.
- `Lab Console`: have a direct console access to the device. The network attributed to your laboratory and the gateway will be indicated.

Below the instance, you can see the practical subject of the lab if there is one.

!!! warning
    Once the leave lab button has been clicked, the lab instance is destroyed and **ALL MODIFICATIONS** made on the lab's devices are erased.

From the instances list tab, you can see all the lab instances that you have joined.

![Screenshot](/images/Students/Student_Instances.png)
By clicking the `Details` button, you get an overview of your instance as follows:

![Screenshot](/images/Students/Student_Instances_Details.png)

You can manage your devices and access information such as the network attributed to the lab instance.

## Lab topology

![Screenshot](/images/Students/Student_lab_topology.png)

You can also see the topology of the lab and access the practical subject and lab details through this screen.

## Remote access

In order to access your lab from your workstation (through SSH, for instance), you need to connect through the RemoteLabz VPN.

To do this, you need to:

1. download a client like <a href="https://openvpn.net/client/">VPNconnect</a> and then install it.
2. click on the OpenVPN file button. An `.ovpn` file will be downloaded.
3. double-click on the downloaded file and VPNConnect will install it automatically.
4. hit the connect button.

If the client displays this, you are connected.

![Screenshot](/images/Students/VPN_Connect.png)
