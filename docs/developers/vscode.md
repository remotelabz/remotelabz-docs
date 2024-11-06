#Code Organization

## Bundle
The remotelabz is divided in 4 parts :

- `remotelabz` for the frontend
- `remotelabz-worker` for the server which execute all VMs, called the backend
- `remotelabz-message-bundle` uses by both remotelabz and remotelabz-worker to manage the message exchanged between the frontend and the backend
- `network-bundle` uses by both frontend and backend


##RemoteLabZ Web Pages Code

The front's webpages code is on ```templates``` directory.

The web page of a lab is on file ```view.html.twig``` and the lab is managed by a react component ```{{ react_component('InstanceManager', {'props': props}) }}``` defined in ```assets/js/components/Instances/```

##RemoteLabZ PHP Code

For the front server and the worker server, the PHP code is located on `/opt/remotelabz/src`.
This project use the Symphony framework,it's sourcefiles are located on `/opt/remotelabz/vendor` .

Remotelabz logs are located on `/opt/remotelabz/var/log`


## Semantic Versioning
Remotelabz version's name follow the rules of semantic versioning.You can find a complete description at : https://semver.org/

## Yarn 

Each time you modify Javascript or ReactJS, you need to do a `yarn encore dev` All JS code is in the asset directory.

## ReactJS

Version 16.8 minimum because some script uses Hooks (https://fr.reactjs.org/docs/hooks-state.html). All codes are not translate to use Hooks.

## Visual Studio Code Installation  
You can use an editor like Visual Studio Code to edit most of the files 

!!! Warning
    Before editing, ensure you have proper permissions and a backup of the files 

To install visual studio according to your system, go to <a href="https://code.visualstudio.com/download">the official download page</a>.

Then double-click on the downloaded file and install it.

Visual studio is supported on Mac,Windows and linux (debian and fedora).

