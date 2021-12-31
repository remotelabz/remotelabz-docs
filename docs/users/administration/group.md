#Group management

A member of a group can be : 

 * simple user
 * administrator

The creator of the group has the administrator role and he can give this role to other members of the group. The administrator of a group manages it. So, it can add members to the group, create a subgroup of this group, change the role of a member, add, or delete a member.

From version 2.4.0, each privileged user of a group (owner or administrator of a group) can add to its group, any laboratories whose he is the author. If a lab is added to a group, each member of any sub-group can also view this lab. But if a laboratory is added only to a sub-group, nobody of its parent group can see this lab.
Let 3 groups, Gr1 with the two following sub-group SubGrA and SubGrB. If Lab A is affected to Gr1, so all members of SubGrA and SubGrB can execute Lab A. But if Lab B is affected to SubGrB, only members of SubGrB will execute this Lab B.

The group has a "visibility" property. It can be 

- public
- private
- internal

All public groups and so, their associated activities, also called labs, can be view by all users of the RemoteLabz.
Internal groups can be view only by the member of the groups and the private groups can be view only by its owner. The administrator of RemoteLabz shows all groups and so, all activities.
The private group can be used to test and develop activities and as soon as the activities are ready, either the owner or the administrator of the group can change the "visibility" property to add the activities in a public or internal groups.