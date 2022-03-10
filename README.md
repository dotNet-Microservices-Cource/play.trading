## Play.Trading
Play Economy Trading microservice

## Build the docker image
```powershell
$version="1.0.3"
$env:GH_OWNER="dotNet-Microservices-Cource"
$env:GH_PAT="[PAT HERE]"
$appname="playeconomyapp"
docker build --secret id=GH_OWNER --secret id=GH_PAT -t "$appname.azurecr.io/play.trading:$version" .
```

## Run the docker image
```powershell
$cosmosDbConnString="[CONN STRING HERE]"
$serviceBusConnString="[CONN STRING HERE]"
docker run -it --rm -p 5006:5006 --name trading -e MongoDbSettings__ConnectionString=$cosmosDbConnString -e ServiceBusSettings__ConnectionString=$serviceBusConnString -e ServiceSettings__MessageBroker="SERVICEBUS" play.trading:$version
```

## Publishing the Docker image
```powershell
az acr login --name $appname
docker push "$appname.azurecr.io/play.trading:$version"
```

## Creating the pod managed identity
```powershell
$resourcegroup="playeconomy"
$managedname="trading"
az identity create --resource-group $resourcegroup --name $managedname
$IDENTITY_RESOURCE_ID=az identity show -g $resourcegroup -n $managedname --query id -otsv

az aks pod-identity add --resource-group $resourcegroup --cluster-name $appname --namespace $managedname --name $managedname --identity-resource-id $IDENTITY_RESOURCE_ID
```

## Granting access to Key Vault secrets
```powershell
$IDENTITY_CLIENT_ID=az identity show -g $resourcegroup -n $managedname --query clientId -otsv
az keyvault set-policy -n $appname --secret-permissions get list --spn $IDENTITY_CLIENT_ID
```

## Creating the Kubernetes resources
```powershell
$namespace="trading"
kubectl apply -f .\kubernetes\trading.yaml -n $namespace
```