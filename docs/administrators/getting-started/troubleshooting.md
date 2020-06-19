# Troubleshooting

## 500 Internal Server Error on Login page
Wrong permission for config/jwt/
```bash
# Change owner of config/jwt/*
chown -R www-data:www-data config/jwt
``` 

## 500 Internal Server Error on Labs page
Wrong permission in var/cache/prod/
```bash
# Change owner of cache/prod/
chown -R www-data:www-data *
```
