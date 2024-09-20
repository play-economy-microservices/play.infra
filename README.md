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