A laboratory is an exercise associated to devices. All devices are configured to the exercise but can be reused to other activities. Only teachers and administrators can create a laboratory using `Labs` menu.

## Lab Creation

To create a lab, first go to the `Labs` tab in the top left corner. Here, you have all the labs that you created.
You'll find the `New Virtual Lab` and `New Physical Lab` buttons in the top right corner. Choose the one that best suits your needs.
 ![Screenshot](/images/Teacher/lab_creation/teacher_home_page.png)

## Virtual Lab 

When you click on `New Virtual Lab`, it opens an editor with an empty canvas. You arrive at the page to build the lab's topology.
![Screenshot](/images/Teacher/lab_creation/teacher_lab_topology.png)
In the sidebar at the left, you have several options.
To create a virtual laboratory, you will need : 

- **Service Container** :  to give to your lab internet connectivity through a DHCP server.
- **Switch Container** : to interconnect every containers in a local network.
- **Containers** : your own containers that you want to use.

### Add an object

Let's start by adding containers to our topology. In the sidebar at the left, we have the first option named `Add object` with which we have :

- **Node**: Add a container, the one you want. 
- **Custom Shape**: Add various shapes with large assets of colors and borders. 
- **Text**: Add text on your topology.

We have also in the side bar in second position `Configured Objects` to check custom shapes.

- **Configured Objects**: View all custom shapes in your topology. You can delete them from this window.

To add our container, we should click on `Add an object` → `Node`. A drop-down menu with all containers will appear. Select the one you need. For this exemple, we choose Natif.

 ![Screenshot](/images/Teacher/first-lab/nodeadd1.png)

On the next screen, you can specify the container's parameters such as the name, the number of processor's core, memory and so on.

![Screenshot](/images/Teacher/first-lab/nodeadd2.png)
![Screenshot](/images/Teacher/first-lab/nodeadd3.png)
![Screenshot](/images/Teacher/first-lab/nodeadd4.png)

Once done, click on the `Save` button. The container will be created and will appear on the top left side of the screen.

![Screenshot](/images/Teacher/first-lab/nodeadd5.png)

To continue our example, we will then repeat the same procedure to add the service container and an Ubuntu container. 

When added, we have to connect those container on the switch. To achieve this, hover on a container and a plug icon will appear. Hold your click on it and wire it to the one called "Natif".

![Screenshot](/images/Teacher/lab_creation/teacher_lab_network_plug.png)

A new screen will show up. Put new network interface on both and save.

![Screenshot](/images/Teacher/lab_creation/teacher_lab_network_connection.png)

This will add a new connection between these two containers. Repeat the same process for each remaining containers. In the end, the laboratory should look like this.

![Screenshot](/images/Teacher/lab_creation/teacher_lab_topology.png)

The laboratory is created and ready to be tested.

### Lab Information and Practical Subject

Prior to testing, let's change the title and the practical subject. To do this, on the sidebar we have the `More Actions` option with: 

- **Edit Practical Subject** : In this section, we add the lab practical subject. We have a Markdown text editor which will allow us to structure it.
- **Edit Lab** : In this window, it is possible to change the title of the lab, its version, its description, the display image and finally the timer which limits the time allotted to the use of this lab.

![Screenshot](/images/Teacher/lab_creation/teacher_more_actions.png)


#### Lab Edition

To change the title, we click on `More Actions` → `Edit Lab`.

![Screenshot](/images/Teacher/lab_creation/teacher_lab_edition.png)

From this menu, you can add an image and change the title of the lab. You can also set a timer. Some elements will be  displayed in the lab's thumbnail.

The result can be previewed in the sidebar option `Lab Details`.

![Screenshot](/images/Teacher/lab_creation/teacher_lab_details.png)


#### Practical Subject Edition

To add a practical subject, we click on `More Actions` → `Edit Practical Subject`.

![Screenshot](/images/Teacher/first-lab/EditPracticalSubject.png)

The subject can be edited via a markdown editor. Links, image and lists can also be added.

The result can be previewed in the sidebar option `Practical Subject`.

![Screenshot](/images/Teacher/lab_creation/teacher_lab_practical_subject.png)

### Testing the Lab
Close the lab by clicking `Close lab` in the sidebar. You will see the newly created lab and you can join it. Click on the `Join lab` button. This will create an instance of your lab. Once done, you will see the list of container associated to your instance.

![Screenshot](/images/Teacher/first-lab/Teacher_lab_start.png)

You can start a container by using the green arrow on the right side of the list. 

1. First, start the Service container
2. Next, the Ubuntu container.

![Screenshot](/images/Teacher/first-lab/Teacher_lab_running.png)

Click on the console icon on the Ubuntu container. Then, `open terminal` and finally you will land on the container's console.

There you can ping google DNS 1.1.1.1, if you got a response it signifies that the DHCP server is working and the lab is fully functional.

![Screenshot](/images/Teacher/first-lab/LabFinalTest.png)


