## Infra
Sample Microservice Shop Infrastructure components

## General Variables
```powershell
$owner="SampleMicroserviceShop"
$appname="microshop"
```

## Add the GitHub package source
```powershell
$owner="SampleMicoserviceShop"
$gh_pat="[PAT HERE]"
dotnet nuget add source --username USERNAME --password $gh_pat --store-password-in-clear-text --name github "https://nuget.pkg.github.com/$owner/index.json"
```

## Creating the Azure resource group
```powershell
az group create --name $appname --location westus2
```

## Creating the CosmosDB account
```powershell
az cosmosdb create --name $appname --resource-group $appname --kind MongoDB --enable-free-tier
```

## Creating the Service Bus namespace
```powershell
az servicebus namespace create --resource-group $appname --name $appname --sku Standard
```

## Register Azure subscription to use AKS
```powershell
az provider register --namespace Microsoft.ContainerService
az provider show -n Microsoft.ContainerService
```

## Creating the AKS cluster
```powershell
az aks create -n $appname -g $appname --node-vm-size Standard_B2s --node-count 2 --attach-acr $appname --enable-oidc-issuer --enable-workload-identity --generate-ssh-keys
az aks get-credentials --resource-group $appname --name $appname
```
