# PDU management

![Screenshot](/images/PDU/PDU_overview.png)

Since RemoteLabz 2.4, it is possible to create and manage virtual PDUs. PDUs are metered, fully manageable power strips commonly used in datacenters.
Each outlet of these devices can be assigned to any device in a physical or logical lab.

!!! warning
    Only an administrator can create or manage PDUs.

## PDU creation

![Screenshot](/images/PDU/newPDU.png)

To create a PDU, click on the `new PDU` button. Then, a new form appears asking for the following information:

- **IP of the PDU**: any IP address, such as 10.0.0.10, for example.
- **Brand**: only APC or Raritan are allowed for now.
- **Model of the PDU**: can be anything.
- **Number of outlets**: number of outlets the PDU will have (0 to 42 max).

Once the PDU is successfully created, it will be listed in the PDU section.

![Screenshot](/images/PDU/PDUadded.png)

## Outlet management

To manage the created PDU, click on its IP address.

![Screenshot](/images/PDU/PDU_overview.png)

The next screen will allow you to:

- modify outlets,
- assign a device to one of its outlets,
- delete an outlet.

## Assign a device to an outlet

![Screenshot](/images/PDU/PDU_overview.png)

To assign a device to an outlet, click on its edit button. A new form appears.

![Screenshot](/images/PDU/EditPDUoutlet.png)

Then, in the list, choose the device you want to assign to the selected outlet and save.

![Screenshot](/images/PDU/EditPDUoutletmodified.png)

Once done, the device is assigned to the selected outlet.
