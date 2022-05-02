# Monitoring_Hands_On

### 0) Prerequisites
The prerequisites from this folder are not required when working in browser via Azure CLI. When working locally you will need the following:

1) Text editor of choice (one with syntax highlighting would be perfect, ex. Visual Studio Code https://code.visualstudio.com/Download )
2) kubectl - https://kubernetes.io/docs/tasks/tools/ (needs to be added to enviromental variables after downloading)
3) helm - https://helm.sh/docs/intro/quickstart (needs to be added to enviromental variables after downloading)

### 1) Logging in to cluster
* Open the Azure CLI in browser
* ```az account set --subscription <subscription_id>```
* ```az aks get-credentials --resource-group <rg_name> --name <cluster_name>```

### 2) To make life a little bit easier set the context in kubectl (so that you don't need to specify the namespace in every single command)
```
kubectl config set-context <context_name_of_your_choice> [--cluster=cluster_nickname] [--user=user_nickname] [--namespace=namespace]
kubectl config use-context <context_name_of_your_choice>
```