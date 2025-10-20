#Code Organization

## Bundle
The remotelabz is divided in 4 parts :

- `remotelabz` for the frontend
- `remotelabz-worker` for the server which execute all VMs, called the backend
- `remotelabz-message-bundle` uses by both remotelabz and remotelabz-worker to manage the message exchanged between the frontend and the backend
- `network-bundle` uses by both frontend and backend

##RemoteLabz

### Web Pages Code

The front's webpages code is on ```templates``` directory.

The web page of a lab is on file ```view.html.twig``` and the lab is managed by a react component ```{{ react_component('InstanceManager', {'props': props}) }}``` defined in ```assets/js/components/Instances/```

### PHP Code

For the front server and the worker server, the PHP code is located on `/opt/remotelabz/src`.

Remotelabz logs are located on `/opt/remotelabz/var/log`


##This Guide

The file used with all the links are defined in

``` bash
docs/mkdocs.yml
```
The other source files are on the `/docs` Directory

The images files are on the `/site/images` Directory

## Code Edition  

### Editor

You can use an editor like Visual Studio Code to edit most of the files 

!!! Warning
    Before editing, ensure you have proper permissions and a backup of the files 

To install visual studio according to your system, go to <a href="https://code.visualstudio.com/download">the official download page</a>.

Then double-click on the downloaded file and install it.

Visual studio is supported on Mac,Windows and linux (debian and fedora).

### Semantic Versioning
Remotelabz version's name follow the rules of semantic versioning.You can find a complete description at : https://semver.org/


## Dependencies

### Yarn 

Each time you modify Javascript or ReactJS, you need to do a `yarn encore dev` All JS code is in the asset directory.

### ReactJS

Version 16.8 minimum because some script uses Hooks (https://fr.reactjs.org/docs/hooks-state.html). All codes are not translate to use Hooks.


### amqp-messenger 

This project uses AMQP messaging protocol to handle communication between the host and the worker.
amqp-messenger is a library required by symphony to handle this protocol.
Recommended version is 5.2.12.

### Symfony

This project is based on Symphony which is a popular MVC framework for PHP.It's sourcefiles are located on `/opt/remotelabz/vendor` .
We recommand version 5.4.2.9 minimum. 


## Device creation workflow


## Laboratory workflow

When a user joins a lab, clones linked to the lab's devices are created. If these devices come from templates with available ISOs, a radio button area will be displayed to select the ISO to boot from.

If the ISO checkbox is checked, the device will boot from the ISO. When the VM is shut down, the export button will be available to save this device as a booted template (which will then have the value 2 in the template), bearing in mind that a template has the value 1 and a linked clone has the value 0.

When a user starts a device, there are three possible scenarios:

 - the device is not associated with an ISO, and we create a clone linked to the template
 - the device can be associated with an ISO and, either :
    - the boot ISO checkbox is checked, so we boot from this iso image
    - or the boot ISO checkbox is not checker, either :
        - there exist a booted template (template value 2), we boot from that image
        - there don't exist a booted template, we start the device's blank image

