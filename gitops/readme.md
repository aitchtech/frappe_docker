```
docker compose --project-name rsis-prod --env-file gitops-rsis-prod/rsis-prod.env -f compose.yaml -f overrides/compose.redis.yaml -f overrides/compose.mariadb.yaml -f overrides/compose.noproxy.yaml config > gitops-rsis-prod/rsis-prod.yaml
```


```
bench new-site --no-mariadb-socket --mariadb-root-password rsisprodDbP@ss --install-app erpnext --admin-password rsisprodP@ss rsis.reshinecarstudio.com
```