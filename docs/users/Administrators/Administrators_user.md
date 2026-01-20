# User management
![Screenshot](/images/Administrator/user-management.png)

By clicking on the  `new user` button, you can create an user. You have to give parameters like :

* **email** 
* **password**
* **last name**
* **first name** 
* **access level** which is the role of the user. 

If you don't want them to log into their account, you have to uncheck the box `Enabled`.

It is also possible to import users via the `import from file` feature. This should be done via a .CSV file, however it must be encoded in UTF-8 and respect the following convention :
```bash
lastname,firstname,email,group
```
## User roles

In RemoteLabz, a user can have one of the following role :
 
 * **student**
 * **teacher**
 * **editor teacher**
 * **administrator**

Permissions by roles are as follows :

![Screenshot](/images/Administrator/Administrator_user_permissions.png)

The administrator can administrate all the application.
