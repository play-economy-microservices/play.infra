# play.infra

Play Economy Infrastructure Components

## Add the GitHub package source

```powershell
$owner="play-economy-microservices"
$gh_pat="[PAT HERE]"

dotnet nuget add source --username USERNAME --password $gh_pat --store-password-in-clear-text --name github "https://nuget.pkg.github.com/$owner/index.json"
```

# Creating the Azure resource group

```powershell
$appname="playeconomy"
az group create --name $appname --location eastus
```

# Creating the Cosmos DB account

```powershell
$dbname="playeconomydb"
az cosmosdb create --name $dbname --resource-group $appname --kind MongoDB --enable-free-tier
```

# Creating the Service Bus namespace

```powershell
$servicebusname="playeconomyservicebus"
az servicebus namespace create --name $servicebusname --resource-group $appname --sku Standard
```

# Creating the Container Registry

```powershell
$containername="playeconomycontainerregistry"
az acr create --name $containername --resource-group $appname --sku Basic
```

# Creating the AKS Cluster

```powershell
$appname="playeconomy"
$containername="playeconomycontainerregistry"
az aks create -n $appname -g $appname --node-vm-size Standard_B2s --node-count 2
--attach-acr $containername --enable-oidc-issuer --enable-workload-identity --generate-ssh-keys

az aks get-credentials --name $appname --resource-group $appname
```

## Creating the Azure Key Vault

```powershell
$appname="playeconomykeyvault"
az keyvault create -n $appname -g $appname
```

## Installing Emissary-ingress

These commands are used to install the CRDs for **Emissary Ingress** into a Kubernetes cluster and then wait for the emissary-apiext deployment to be fully available.

```powershell
$namespace="emissary"
$appname="playeconomy-azure-emissary"

helm repo add datawire https://app.getambassador.io
helm repo update

kubectl apply -f https://app.getambassador.io/yaml/emissary/3.9.1/emissary-crds.yaml
kubectl wait --timeout=90s --for=condition=available deployment emissary-apiext -n emissary-system
```

## Configure Emissary Ingress Routing
```powershell
$namespace="emissary"
$appname="playeconomy-azure-emissary"

kubectl apply -f ./emissary-ingress/listener.yaml -n $namespace
kubectl apply -f ./emissary-ingress/mappings.yaml -n $namespace
```