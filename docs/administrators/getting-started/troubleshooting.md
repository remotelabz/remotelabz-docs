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

## Error in prod when update
If an error occurred when you update your version, from prod or older version, you can :

 - delete the `*.lock` file
 - check with `git status` if you have additional file on your filesystem and delete them
 - delete the `var/cache` directory