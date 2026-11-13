# Laboratory management

## Basic concepts

A laboratory, also called in this documentation an activity, is an exercise associated with devices. All devices are configured for the exercise, but they can be reused for other activities.

## Instance management

In RemoteLabz, when you join a lab, you create an instance of this lab. You can manage its associated containers through the laboratory management panel.

![Screenshot](/images/Administrator/Administrator_instance_management.png)

To manage an instance, click on the `Details` button.

![Screenshot](/images/Teacher/Teacher_lab_instance_details.png)

From there, you have access to all the information about the instance. You can:

- See the owner of the instance
- Start all, stop all and reset all devices
- Start, stop or reset one device
- Leave the lab

!!! warning
    Once the leave lab button has been clicked, the lab instance is destroyed and **ALL MODIFICATIONS** made on the lab's containers are erased.

## Lab topology

![Screenshot](/images/Teacher/labtopology.png)

A laboratory basically needs three elements:

- **Service Device**: to give your lab internet connectivity through a DHCP server. It should always be started first.
- **Switch**: to interconnect all the containers in a local network.
- **Containers**: your own containers.

With the sidebar on the left, you can also access the subject and lab title through this screen.

!!! warning
    Only teachers and administrators can create a laboratory using the `new lab` button.

## Lab edition

![Screenshot](/images/Teacher/Teacher_Lab_edition.png)

To edit a laboratory, if you joined the lab you should leave it first. This action will destroy your instance and all associated containers.

!!! warning
    You can't edit a lab once it has been joined. If users have already joined your lab, they must leave it to get the newly edited lab. This will cause the loss of all their data and containers.

Once the laboratory has been left, an `edit` button appears on the top right corner, which will lead you to the lab editor.
From there, you can modify every aspect of the laboratory, including its title, topology, practical subject and so on.

You can also delete the lab from there by clicking on the `delete` button.

## Lab export

It is also possible to save the lab topology on your computer. This can be accomplished through the `export` button.

A `tar.gz` file with the lab's JSON file and the qcow2 image of the container will be downloaded. You can then import the lab.

!!! warning
    This only saves the topology, not the lab's data on the containers!

## Lab creation

![Screenshot](/images/Teacher/Teacher_lab_tab.png)

It is possible to create a new lab on RemoteLabz with the `New Virtual Lab` and `New Physical Lab` buttons that will appear once you get to the lab section.

You can also import a previously exported lab from there through the `import` button.

You can have more detailed information in the [Laboratory Creation](Administrator_lab_creation.md) section.

## Remote access

In order to access your lab from your workstation (through SSH, for instance), you need to connect through the RemoteLabz VPN.

To do this, you need to:

1. download a client like <a href="https://openvpn.net/client/">VPNconnect</a> and then install it.
2. click on the OpenVPN file button. An `.ovpn` file will be downloaded.
3. double-click on the downloaded file and VPNConnect will install it automatically.
4. hit the connect button.

If the client displays this, you are connected.

![Screenshot](/images/Teacher/VPN_Connect.png)
