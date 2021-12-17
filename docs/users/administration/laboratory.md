A laboratory, also called in this documentation an activity, is an exercise associated to devices. All devices are configured to the exercise but can be reused to other activities.
Only teachers and administrators can create a laboratory using "labs" menu.

# Configure a laboratory

When you add a device, in its default configuration, it has no network interface. You have to add a network interface to be able to configure it from the laboratory. Each network interface can be configure on a vlan. Default vlan is 0. In this setting, no vlan is configured on the hidden switch with connected all devices of the laboratory.

# The svc device ?

When you create a laboratory, a device called "svc" is automatically added. The device is a "service" device. It is configured with a DHCP server and it is automatically connected to the internal switch.


