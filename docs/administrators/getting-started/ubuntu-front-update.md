# RemoteLabz's front update guide

!!! warning
    Documentation in progress

##To version 2.X.Y
You have to be connected on your server with sudo's rights

###Save your .env file
If you don't have a .env.local, save your .env to .env.local
```bash
sudo -u remotelabz cp .env .env.local
```

###Update process
```bash
cd /opt/remotelabz
sudo -u remotelabz git pull
sudo -u remotelabz git checkout tags/2.X.Y
Compare .env.dist et .env si présence de nouvelle variable
php bin/console doctrine:migrations:migrate
composer install
yarn install
yarn encore prod
php bin/console cache:clear
sudo chown remotelabz:www-data * -R
sudo chmod g+w /opt/remotelabz/var/cache * -R
sudo chmod g+r config/jwt/private.pem
```