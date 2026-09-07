# Schedule Actions

You can schedule to start and stop a lab for a group at a specified date and time. 
This page is divided into two sections:

- **New scheduled action** — it is used to create a new scheduled action.
- **My scheduled actions** — it lists the actions you have already scheduled.

![Screenshot](/images/scheduled_actions/scheduled_actions_overview.png)

## Create a scheduled action

To create a scheduled action, fill in the form with the following fields:

- **Group** Select the group for which the action will be scheduled. Only the groups you manage are listed.
- **Lab** Select the lab to which the action applies. 
- **Start date**  Set the date and time at which the action will start. All devices in the lab will be started at this date and time.
- **End date** *(optional)* Set the date and time at which the end action will be triggered. Leave this field empty if you do not want an automatic end action.
- **End action**  Choose what happens when the end date is reached:
    - Stop — Stops all devices. Instances are not destroyed and can be restarted later.
    - Leave — Deletes all instances. Instances are permanently destroyed.

Once all fields are filled in, click Schedule to confirm and create the scheduled action.

## View scheduled action

You have the `My scheduled actions` panel, on the right side of the page. It displays the list of actions you have scheduled with a counter showing how many are pending. You can delete the scheduled actions by clicking the X button.s 

If no action has been scheduled yet, the panel displays: "No scheduled action yet."