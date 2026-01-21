# Laboratory management

## Basic concepts

A laboratory, also called in this documentation an activity, is an exercise associated to devices. All devices are configured to the exercise but can be reused to other activities.

## Instance management

In RemoteLabz, when you join a lab, you create an instance of this lab. You can manage its associated containers through the laboratory management panel.

![Screenshot](/images/Administrator/Administrator_instance_management.png)

To manage an instance, click on the `Details` button.

![Screenshot](/images/Teacher/Teacher_lab_instance_details.png)

From there, you have access to all informations about the instance, you can :

- See the owner of the instance
- Start all, stop all and reset all devices
- Start, stop or reset one device
- Leave the lab


!!!Warning
     Once the leave lab button has been clicked, it destroy the lab instance erasing **ALL MODIFICATIONS** on the lab's containers.

## Lab Topology

![Screenshot](/images/Teacher/labtopology.png)

A laboratory needs basically three elements:

- **Service Device** : to give to your lab internet connectivity through a DHCP server. It should be always started it first.
- **Switch** : to interconnect every containers in a local network.
- **Containers** : your own containers.

With the sidebar on the left, you can access to the subject and lab title through this screen as well.
 
!!!Warning
    Only teachers and administrators can create a laboratory using the `new lab` button.

## Lab Edition

![Screenshot](/images/Teacher/Teacher_Lab_edition.png)

To edit a laboratory, if you joined the lab you should leave it. This action will destroy your instance and all associated containers.

!!!Warning
    You can't edit the lab once it has been joined. If users have already joined your lab, to have the new edited lab they should leave it. This will cause the loss of all their data and containers.

Once the laboratory has been left, an `edit` button appears on the top right corner which will lead you to the lab editor.
From there, you can modify every aspects of the laboratory including its title, topology, practical subject...

You can also delete the lab from there through clicking on the `delete` button.

## Lab Export 

It is also possible to save the lab topology on your computer. This can be accomplished through the `export` button. 

A `tar.gz` file with the lab's json file and the qcow2 image of the container will be downloaded. You can then import the lab.

!!!Warning
    This saves only the topology not the lab's data on containers !

## Lab Creation

![Screenshot](/images/Teacher/Teacher_lab_tab.png)

It is possible to create a new lab on RemoteLabz with the `New Virtual Lab` and `New Physical Lab` buttons that will appear once you get to the lab section.

You can also import a previously exported lab from there through the `import` button.

You can have more precise information in the `Laboratory Creation` section
## Remote access

In order to access to your lab from your workstation through SSH for instance, you need to connect through RemoteLabz VPN.

To do this, you need to:

1. download a client like <a href="https://openvpn.net/client/">VPNconnect</a> and then install it.
2. click on the OpenVPN file button.An .ovpn file will be downloaded.
3. double-click on the downloaded file and VPNConnect will install it automatically.
4. hit the connect button.

If the client display this, you are connected.

<img src="/images/Teacher/VPN_Connect.png" height=600px width=300px>



