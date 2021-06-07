# RemoteLabz's front update guide

!!! warning
    Documentation in progress

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
Ou bien la version en cours de développement :
```bash
git checkout dev
```

Compare `.env.dist` to be sure we have all variables in your `.env` or `.env.local` file 

```bash
sudo composer install
sudo php bin/console cache:clear
sudo chown remotelabz:www-data * -R
sudo chmod g+w /opt/remotelabz/var/cache * -R
sudo chmod g+r config/jwt/private.pem
```