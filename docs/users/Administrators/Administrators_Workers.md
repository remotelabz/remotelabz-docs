#Worker configuration
In remotelabz 2.4.4 it is possible to add workers without modifying the `messenger.yaml` file.
However this is restricted to administrators.

##Add workers

To do this, go to Admin area -> System -> Worker configuration.

![Screenshot](/images/workers/Worker-summary.png)

Then click on the `+` button and enter the worker IP address.
![Screenshot](/images/workers/Worker_add.png)

When done, the new worker is added and you can check his status on the Usage tab.

##Modify workers

It is also possible to disable or delete any RemoteLabZ workers from there.

!!!Warning
    Disabling a worker can corrupt or destroy lab instances.It is strongly recommended to leave every running labs on the selected worker before shutting it down.
 





