# Services management

![Screenshot](/images/Administrator/Service/Administrator_service.png)

From this area, you can manage (start/stop) the essential RemoteLabz services such as:

* **Local services**:
    * **Messaging services**: manage the RabbitMQ service, which handles communications between the front and the worker.
    * **Proxy services**: manage the proxy service, which handles the WebSocket connections (consoles and VNC) between the front and the worker.
    * **Router services**: manage routes to each laboratory.

* **System Checks**:
    * **SSH Connection Status**: check the SSH connectivity to all workers.
    * **Certificate Status**: check the validity of SSL/TLS certificates.

* **Remote services**:
    * **Worker services**: manage the worker services.
