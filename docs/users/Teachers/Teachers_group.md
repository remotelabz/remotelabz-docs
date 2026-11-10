# Group management

## Group creation

To create a group, click on the `Groups` tab at the top left corner.

![Screenshot](/images/Teacher/group/Teacher_new_group.png)

The creator of the group has the administrator role, and they can give this role to other members of the group.

The group has a `visibility` property. It can be:

- **public**: all associated activities can be viewed by all users of RemoteLabz.
- **private**: all associated activities can only be viewed by members of the group.
- **internal**: all associated activities can only be viewed by the owner of the group.

The administrator of RemoteLabz sees all groups and, so, all activities.

Internal groups can be used to test and develop activities. As soon as the activities are ready, either the owner or the administrator of the group can change the `visibility` property to move the activities into a public or private group.

!!! tip
    From version 2.4.0, each privileged user of a group (owner or administrator) can add to their group any laboratory for which they are the author. If a lab is added to a group, each member of any sub-group can also view this lab. But if a laboratory is added only to a sub-group, nobody in its parent group can see this lab. Let's take 3 groups: Gr1, with the two following sub-groups SubGrA and SubGrB. If Lab A is added to Gr1, then all members of SubGrA and SubGrB can execute Lab A. But if Lab B is added only to SubGrB, only members of SubGrB will be able to execute Lab B.

In the group setting panel, every parameter including name, subname, visibility and description can be changed by the group's administrator(s). It is also possible to delete the group or change its namespace from there.

## Adding users to a group

![Screenshot](/images/Teacher/group/Teacher_add_user.png)

A member of a group can have one of two profiles:

* **user**: has limited privileges; it can only view other users and interact with laboratories.
* **administrator**: manages the group. So, it can add members to the group, create a subgroup of this group, change the role of a member, or delete a member.

You can add each user one by one by searching their name or email address and giving them a role.
It is also possible to add multiple users to a group via a .CSV file; however, it must be encoded in UTF-8 and respect the following convention.

```bash
numero;surname;name;email
```

It is also possible to import users from another group via the `import from group` feature.

## Adding laboratories to a group

![Screenshot](/images/Teacher/group/Teacher_add_lab.png)

If you are an administrator of a group, you can add any lab to it. In the sidebar on the left, click on `Labs`. Search for your laboratory by its name and add it. This will allow every registered user in this group to see the lab and start their own instance of it.

## Delete a group

![Screenshot](/images/Teacher/group/Teacher_Group_delete.png)

Only the owner can delete a group. Once deleted, the group cannot be restored. To delete a group, go to the group `Settings` and, at the very bottom, click on the `Delete` button.

## Change namespace

![Screenshot](/images/Teacher/group/Teacher_Group namespace.png)

Only the owner can change the namespace of a group. To do this, go to the group `Settings` and then `change namespace`. From there, you can:

- Select the namespace of another group. Once changed, the group will become a subgroup of the selected group.
- Leave the namespace empty. This will clear the namespace. Your group will remain as it is. If it is in a subgroup, it will become an independent group.

!!! warning
    This may delete all users (except the owner) or laboratories from the group.
