# RemoteLabz's front update guide

###Save your .env file
All configuration variables are saved in the file `.env.local` or if you don't have this file, in the file `.env`
If you don't have a .env.local, save your .env to .env.local
```bash
sudo -u remotelabz cp .env .env.local
```

###Update the code
```bash
cd /opt/remotelabz
git fetch
git pull
```

!!! info
    To choose a specific version :
    ```bash
    git checkout tags/2.X.Y
    ```
    or a branch
    ```bash
    git checkout dev
    ```

Compare your `.env.local` file with the `.env` file to be sure we have the same parameters defined 

```bash
sudo composer update
sudo yarn encore prod
sudo php bin/console doctrine:migrations:migrate
sudo php bin/console cache:clear
sudo chown remotelabz:www-data * -R
sudo chmod g+w /opt/remotelabz/var -R
sudo chmod g+r config/jwt/private.pem
sudo systemctl daemon-reload
sudo service remotelabz restart
```

!!! info
    To update from 2.3 to 2.4, you can add default image to all labs with the following command :
    ```bash
    sudo find /opt/remotelabz/public/uploads/lab/banner/* -type d -exec cp /opt/remotelabz/public/build/images/logo/nopic.jpg {}/nopic.jpg \;
    ```
    You also have to add all user in the default group if you want they can execute some basic labs
