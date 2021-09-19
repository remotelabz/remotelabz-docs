# RemoteLabz's front update guide

###Save your .env file
All configuration variables are saved in the file `.env.local` or if you don't have this file, in the file `.env`
If you don't have a .env.local, save your .env to .env.local
```bash
sudo -u remotelabz cp .env .env.local
```

###Update the code
```bash
cd /opt/remotelabz-worker
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
sudo php bin/console cache:clear
sudo chown remotelabz-worker:www-data * -R
sudo chmod g+w /opt/remotelabz-worker/var -R
sudo systemctl daemon-reload
sudo service remotelabz-worker restart
```