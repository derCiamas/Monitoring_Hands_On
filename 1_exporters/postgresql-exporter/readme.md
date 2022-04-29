1) Pull image:
```
docker pull prometheuscommunity/postgres-exporter:latest
```
2) Retag: 
```
docker tag prometheuscommunity/postgres-exporter:latest <yourContainerRegistry>.azurecr.io/postgres-exporter:latest
```
3) Login to Azure Container Registry
```
az acr login
```
4) Push image to ACR: 
```
docker push <yourContainerRegistry>.azurecr.io/postgres-exporter:latest
```
5) Create PostgreSQL user for your monitoring purposes, add its name to ***values.yaml*** under ***dbSettings.dbusername***
6) Add secret named "postgresql-exporter-user-pass" to your Azure Key Vault and store there password for the user created in the previous point
7) Update values in ***values.yaml***
8) Navigate to ***exporters*** directory
9) Install chart:
```
helm install postgresql-exporter postgresql-exporter
```


Adding custom queryies:
1) Add to extensions/extendedqueries.yaml
2) WARNING: removing from extendedqueries.yaml does not instantly remove the metric from Grafana