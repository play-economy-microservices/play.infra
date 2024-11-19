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

## Installing Cert-Manager

```powershell
$namespace="emissary"

helm repo add jetstack https://charts.jetstack.io --force-update
helm repo update
helm install cert-manager jetstack/cert-manager --version v1.16.1 --set crds.enabled=true --namespace $namespace
```

## Creating the Cluster Issuer

```powershell
$namespace="emissary"

kubectl apply -f ./cert-manager/cluster-issuer.yaml -n $namespace
kubectl apply -f ./cert-manager/acme-challenge.yaml -n $namespace
```

## Creating the TLS Certificate

```powershell
$namespace="emissary"

kubectl apply -f ./emissary-ingress/tls-certificate.yaml -n $namespace
```

## Enabling TLS and HTTPS

This will be used for the gateway to belistened by the outside world.

```powershell
$namespace="emissary"

kubectl apply -f ./emissary-ingress/host.yaml -n $namespace
```

## Packaging and publishing the microservice Helm Chart

```powershell
$appname="playeconomyacr"

# package the helm chart
helm package ./helm/microservice

$helmUser=[guid]::Empty.Guid
$helmPassword=az acr login --name $appname --expose-token --output tsv --query accessToken

# This is no longer needed after Helm v3.8.0
$env:HELM_EXPERIMENTAL_OCI=1

# authenticate
helm registry login "$appname.azurecr.io" --username $helmUser --password $helmPassword

# push to registry
helm push microservice-0.1.1.tgz oci://$appname.azurecr.io/helm
```

## Create Github service principal

```powershell
$appname="playeconomy"
$subId="755900f3-695f-4e79-9b12-1cc2b93e76ae"

$appId = az ad sp create-for-rbac -n "GitHub" --query appId --output tsv

az role assignment create --assignee $appId --role "AcrPush"  --scope /subscriptions/$subid/resourceGroups/$appname/providers/Microsoft.ContainerRegistry/registries/$acrName

az role assignment create --assignee $appId --role "Azure Kubernetes Service Cluster User Role" --scope /subscriptions/$subId/resourceGroups/$appname/providers/Microsoft.ContainerService/managedClusters/$appname

az role assignment create --assignee $appId --role "Azure Kubernetes Service Contributor Role" --scope /subscriptions/$subId/resourceGroups/$appname/providers/Microsoft.ContainerService/managedClusters/$appname
```

## Deploying Seq to AKS

```powershell
# Add the repository where the chart is stored.
helm repo add datalust https://helm.datalust.co
helm repo update

# Installs the seq chart from the datalust repository into your K8s Cluster.
helm install seq datalust/seq -n observability --create-namespace
```
