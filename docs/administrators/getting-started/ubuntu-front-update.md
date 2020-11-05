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
git pull
```

Pour choisir une version précise :
```bash
git checkout tags/2.X.Y
```
Ou bien la version en cours de dévellopement :
```bash
git checkout dev
```

Compare `.env.dist` to be sure we have the rigth and all variable in the `.env` file 

```bash
sudo composer install
sudo yarn install
sudo yarn encore prod
sudo php bin/console doctrine:migrations:migrate
sudo php bin/console cache:clear
sudo chown remotelabz:www-data * -R
sudo chmod g+w /opt/remotelabz/var/cache * -R
sudo chmod g+r config/jwt/private.pem
```