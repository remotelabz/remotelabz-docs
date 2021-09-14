# Bundle
The remotelabz is divided in 4 parts :

- `remotelabz` for the frontend
- `remotelabz-worker` for the server which execute all VMs, called the backend
- `remotelabz-message-bundle` uses by both remotelabz and remotelabz-worker to manage the message exchanged between the frontend and the backend
- `network-bundle` uses by both frontend and backend

# Certificates generation for local development

To develop on own machine, we need to build certificate to support both http and https protocols :

Generate the key for the server certificate
``` bash
openssl genrsa 2048 > server-dev-rlz-ca.key
```

Generate the CA certificate
``` bash
openssl req -new -x509 -days 365 -key server-dev-rlz-ca.key -out server-dev-rlz-ca.crt
```

Generate the key for the server certificate
``` bash
openssl genrsa 2048 > server-dev-rlz.key
```

Create the CSR
``` bash
openssl req -new -key server-dev-rlz.key -out server-dev-rlz.csr
```

Sign the certificate of the server: (do not forget to give the IP of your web server)
``` bash
openssl x509 -req -in server-dev-rlz.csr -out server-dev-rlz.crt -CA server-dev-rlz-ca.crt -CAkey server-dev-rlz-ca.key -CAcreateserial -CAserial server-dev-rlz-ca-serial.srl
```

# Code organisation
The front's code is on ```templates``` directory.

The web page of a lab is on file ```view.html.twig``` and the lab is managed by a react component ```{{ react_component('InstanceManager', {'props': props}) }}``` defined in ```assets/js/components/Instances/```

When a user starts a lab, a message (json) is sent to the worker and analyzed by ```MessageHandler/InstanceActionMessageHandler.php```. The instance, which is a started lab, is managed by the Instance Manager service in ```Service/Instance/InstanceManager.php```

# Semantic Versioning

https://semver.org/
