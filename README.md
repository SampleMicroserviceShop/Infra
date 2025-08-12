## Infra
Sample Microservice Shop Infrastructure components

## General Variables
```powershell
$owner="SampleMicroserviceShop"
$appname="microshop"
$emissary_namespace="emissary"
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

## Register Azure subscription to use ACR
```powershell
az provider register --namespace Microsoft.ContainerRegistry
```

## Creating the container registry
```powershell
az acr create --name $appname --resource-group $appname --sku Basic
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

## Creating the Key Vault
```powershell
az keyvault create -n $appname -g $appname
```

## Installing Emissary-ingress - Add the Repo:
```powershell
helm repo add datawire https://app.getambassador.io
helm repo update
```
 
## Installing Emissary-ingress - Create Namespace and Install:
```powershell
kubectl create namespace $emissary_namespace
kubectl apply -f https://app.getambassador.io/yaml/emissary/3.9.1/emissary-crds.yaml
 
kubectl wait --timeout=90s --for=condition=available deployment emissary-apiext -n emissary-system

helm install emissary-ingress --namespace $emissary_namespace datawire/emissary-ingress --set service.annotations."service\.beta\.kubernetes\.io/azure-dns-label-name"=$appname 
kubectl -n emissary wait deploy -l app.kubernetes.io/instance=emissary-ingress --for=condition=available --timeout=90s
```

## Configuring Emissary-ingress routing
```powershell
kubectl apply -f .\emissary-ingress\listener.yaml -n $emissary_namespace
kubectl apply -f .\emissary-ingress\mappings.yaml -n $emissary_namespace
```

## Installing cert-manager
```powershell
helm install cert-manager oci://quay.io/jetstack/charts/cert-manager --version v1.18.2 --namespace $emissary_namespace  --create-namespace --set crds.enabled=true
```

## Creating the cluster issuer
```powershell
kubectl apply -f .\cert-manager\cluster-issuer.yaml -n $emissary_namespace
kubectl apply -f .\cert-manager\acme-challenge.yaml -n $emissary_namespace
```

## Creating the tls certificate
```powershell
kubectl apply -f .\emissary-ingress\tls-certificate.yaml -n $emissary_namespace
```
