# User management
![Screenshot](/images/Administrator/user-management.png)

## User creation

By clicking on the  `new user` button, you can create an user. You have to give parameters like :

* **email** 
* **password**
* **last name**
* **first name** 
* **access level** which is the role of the user. 

If you don't want them to log into their account, you have to uncheck the box `Enabled`.

## User import

It is also possible to import users via the `import from file` feature. This should be done via a .CSV file, however it must be encoded in UTF-8 and respect the following convention :
```bash
lastname,firstname,email,password,group
```
The password and the group are optional fields. However, if the provided password **is not strong enough** (minimum 12 chars, uppercase, lowercase, digit, special char (!@#$%&^\*)), a new password **will be generated automatically** and will be displayed after the import.

The user can be notified of their password by email : you can check the `Send password by email` box.

* **Checked:** All imported users will receive their password by email.
* **Unchecked:** No emails will be sent, passwords will only be displayed on the results page.

## User roles

In RemoteLabz, a user can have one of the following role :
 
 * **student**
 * **teacher**
 * **editor teacher**
 * **administrator**

Permissions by roles are as follows :

![Screenshot](/images/Administrator/Administrator_user_permissions.png)

The administrator can administrate all the application.
