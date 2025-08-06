## Infra
Sample Microservice Shop Infrastructure components

## General Variables
```powershell
$owner="SampleMicroserviceShop"
$appname="MicroShop"
```

## Add the GitHub package source
```powershell
$owner="SampleMicoserviceShop"
$gh_pat="[PAT HERE]"
dotnet nuget add source --username USERNAME --password $gh_pat --store-password-in-clear-text --name github "https://nuget.pkg.github.com/$owner/index.json"
```

## Creating the Azure resource group
```powershell
az group create --name $appname --location eastus
```

## Creating the CosmosDB account
```powershell
az cosmosdb create --name $appname --resource-group $appname --kind MongoDB --enable-free-tier
```

## Creating the AKS cluster
```powershell
az feature register --name EnablePodIdentityPreview --namespace Microsoft.ContainerService
az extension add --name aks-preview
az aks create -n $appname -g $appname --node-vm-size Standard_B2s --node-count 2 --attach-acr $appname --enable-pod-identity --enable-workload-identity --generate-ssh-keys
az aks get-credentials --resource-group $appname --name $appname
```
