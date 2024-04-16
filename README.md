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
$dbname="playeconomydb"
az group create --name $appname --location eastus
```

# Creating the Cosmos DB account
```powershell
az cosmosdb create --name $dbname --resource-group $appname --kind MongoDB --enable-free-tier
```
