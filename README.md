# Infra

This is the infrasturcuture for the Economy Transaction System.

![Alt Text](https://i.imgur.com/ps4p23V.png)

![](https://i.imgur.com/JfRApwj.png)

![](https://i.imgur.com/2yRVIE6.png)

![](https://i.imgur.com/WpJoAXJ.png)

![](https://i.imgur.com/rABLadh.png)

# How to run the system locally

**Requirements**:

- .NET 7+ SDK
- VS Code or JetBrains Rider
- Docker (Optional)

**Clone all repositories**:

- Catalog
- identity
- Inventory
- Trading
- Frontend

Start `Infra` components first:

```ps
docker compose up -d
```

Start the `Identity` service:

1. Add secret key to your service in your `.csproj` file.
2. Ensure you trust the Identity local certificates first:

```
dotnet dev-certs https --check
dotnet dev-certs https --trust
```

3. Run the service:

```powershell Powershell
dotnet run
```

Start the `Catalog` service:

```powershell Powershell
dotnet run
```

Start the `Inventory` service:

```
dotnet run
```

Start the `Frontend` service:

```
npm i

npm start
```

## Testing the system:

By default an admin user is created by the `Identity` service:

- Username: admin@play.com
- Password: Pass@word1

1. Test the system through **Postman** first:

- identityBaseUrl: `https://localhost:5003`
- CatalogBaseUrl: `https://localhost:5001`
- TradingBaseUrl: `https://localhost:5007`

You will need to get an access token to call any service.

## Identity

```js
GET {{identityBaseUrl}}/users
```

Have the following settings set up in your **Postman** **Authorazation** section:

- Auth Type: oAuth 2.0
- Use Token Type: Access Token
- Header Prefix: Bearer
- Grant type: Authorization Code (With PKCE)
- Callback URL: `urn:ietf:wg:oauth:2.0:oob`
- Auth URL: `{{identityBaseUrl}}/connect/authorize?prompt=login`
- Access Token URL: `{{identityBaseUrl}}/connect/token`
- Client ID: Postman
- Code Challenge Method: `SHA-256`
- Scope: `openid profile IdentityServerApi`
- Client Authentication: Send as Basic Auth Header

**NOTE**: Any other option I didn't list, leave that blank or unchecked.

Then click on **Get new access token**. After getting the access token, send a GET request.

To test other `Identity` endpoints:

```js
GET {{identityBaseUrl}}/users/:id
PUT {{identityBaseUrl}}/users/:id
DELETE {{identityBaseUrl}}/users/:id

GET {{identityBaseUrl}}/.well-known/openid-configuration

GET {{identityBaseUrl}}/health/live
```

You can then create a normal user that will be used on other services. You may want to first test using the **admin** user first though on all services.

## Catalog

**NOTE**: Authorization scopes vary on endpoint.

**POST**

- Scope: `openid profile catalog.fullaccess catalog.writeaccess roles`

```js
POST {{catalogBaseUrl
}}/items
```

```json
{
  "name": "Potion",
  "description": "Restores a small amount of HP",
  "price": 5
}
```

**GET**

- Scope: `openid profile catalog.readaccess`

```js
GET {{catalogBaseUrl
}}/items

GET {{catalogBaseUrl}}/items/:itemId
PUT {{baseUrl}}/items/:itemId

GET {{catalogBaseUrl
}}Url
}}/health/live
```

## Trading

**POST**

- Scope: `openid profile catalog.fullaccess inventory.fullaccess trading.fullaccess IdentityServerApi roles`

```js
POST {{tradingBaseUrl}}/purchase
```

```json
{
  "ItemId": "<itemId>",
  "Quantity": 1,
  "IdempotencyId": "{{$guid}}"
}
```

**GET**

- Scope: `openid profile trading.fullaccess`

## Inventory

**GET**

- Scope: `openid profile inventory.fullaccess`

```js
GET {{inventoryBaseUrl}}/items?userId=<userId>
```

**POST**
Scope: `{{inventoryBaseUrl}}/items/`

```json
{
  "userId": "<userId>",
  "catalogItemId": "<catalogItemId>",
  "quantity": 3
}
```

## Add the GitHub package source

```powershell
$owner="play-economy-microservices"
$gh_pat="[PAT HERE]"

dotnet nuget add source --username USERNAME --password $gh_pat --store-password-in-clear-text --name github "https://nuget.pkg.github.com/$owner/index.json"
```

## Creating the Azure resource group

```powershell
$appname="playeconomy"

az group create --name $appname --location eastus
```

## Creating the Cosmos DB account

```powershell
$dbname="playeconomydb"

az cosmosdb create --name $dbname --resource-group $appname --kind MongoDB --enable-free-tier
```

## Creating the Service Bus namespace

```powershell
$servicebusname="playeconomyservicebus"

az servicebus namespace create --name $servicebusname --resource-group $appname --sku Standard
```

## Creating the Container Registry

```powershell
$containername="playeconomycontainerregistry"

az acr create --name $containername --resource-group $appname --sku Basic
```

## Creating the AKS Cluster

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
