# Installation
## General informations

### From outside to RemoteLabz
- TCP 80, 443 : http(s) pages
- TCP 8000 : websocket
- UDP 1194 : OpenVPN

### From RemoteLabz to RemoteLabz Worker
- TCP 8080 : Remotelabz-Worker Internal API

### From RemoteLabz Worker to Remotelabz
- TCP 5672 : RabbitMQ

### Logs
- Logs are located under `/opt/remotelabz/var/log/`
