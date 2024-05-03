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