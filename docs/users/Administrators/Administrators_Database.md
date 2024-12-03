#Database backup
In RemotelabZ 2.4.4, it is possible for administrators to create database backups.They can also import them in case of database corruption or inconsistencies... 
![Screenshot](/images/Administrator/Database_backup/backup_summary.png)

##Backup Creation

This can be achieved by clicking on `Make a backup button`. Once done it will download an zip archive containing pictures, banners and a full dump of the database.The zip has as default name the backup's date. 

![Screenshot](/images/Administrator/Database_backup/backup_files.png)

Once created it will be listed on the page and available to download again if needed.

![Screenshot](/images/Administrator/Database_backup/backup_added.png)

##Import an existing Backup.

You can import a existing backup by clicking on the `import` button.By doing this, a form will appear asking you to provide a backup zip archive.

![Screenshot](/images/Administrator/Database_backup/backup_import_form.png)

Once done click on `Submit` button and then the whole database will be reverted to the same state as the one from the submitted zip backup.

!!!Warning 
    Modifying the database can lead to permanent data deletion (such as user, groups , labs, lab instance...).Always make a backup prior to import another one. 
