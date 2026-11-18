# Sandbox

The `Sandbox` menu allows you to modify the template of a device.

![Screenshot](/images/Teacher/Teacher_Sandbox.png)

## Sandbox modification

These templates are the devices used when creating a virtual laboratory.
As soon as you choose to modify an existing template, a new Sandbox lab is created. In this lab, you will only see the device on which you want to make modifications, along with a DHCP device.

1. Start the DHCP_Service device first, to start a DHCP service for your lab.
2. Next, start the device you want to modify.

!!! note
    The device must be started at least once before it can be exported.

## Sandbox export

![Screenshot](/images/Teacher/Teacher_sandbox_export.png)

To make an export, you have to:

1. Do your modifications.
2. Stop the device.
3. Click on `Export`, choose a name for your new device template, then click on `Export device`.

After you have clicked on the `Export Device` button, wait until the process is complete. Once done, the process automatically leaves the lab and the corresponding new device template and operating system are created.

!!! warning
    Only teachers and administrators can access this menu and modify an existing default image.

!!! info
    It is recommended to perform a manual shutdown of your device to be sure that the write cache is flushed.
