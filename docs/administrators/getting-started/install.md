# Installation
## General informations
The remotelabz need at least 2 hosts :
 
 - 1 for the web site, called the RemoteLabz's front.
 - 1 for the server which execute the virtuals machines, called worker.
 - (Optionnal) 1 for the RabbitMQ which can be also a virtual machine.

The front can be a virtual machine and it will be the only host with an access from Internet. So, you have to secure your ssh service with the lastest recommandation and we recommand you to install a fail2ban service for example.

![network_architecture](/images/RemoteLabz-Ports.png)

## Ports used

### From outside to RemoteLabz
- TCP 80, 443 : http(s) pages
- TCP 8000 : WebSocket
- UDP 1194 : OpenVPN

### From RemoteLabz to RemoteLabz Worker
- TCP 8080 : Remotelabz-Worker Internal API

### From RemoteLabz Worker to RabbitMQ
- TCP 5672 : AMQP

### Logs
- RemoteLabz's logs are located under `/opt/remotelabz/var/log/` and RabbitMQ's log under `/var/log/rabbitmq`
