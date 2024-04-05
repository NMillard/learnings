# Container apps

Setup your environment by installing the Container Apps extensions.

1. Run `az extension add --name containerapp --upgrade`
2. and then add the namespace `az provider register --namespace Microsoft.App`, `az provider register --namespace Microsoft.OperationalInsights`


## Create
You'll first need to have a resource group that all the resources, such as identities, containerapp environment, and container apps will reside in.  

`az group create --name $RG --location westeurope`


## Interacting with resources
Create a managed identity and assign it to the resources that should communicate. This may for instance be a containerapp retrieving blobs from storage account or keys from a KeyVault.

Create an identity:  
`az identity create --resource-group $RG --name $IDENTITY_NAME`

Export the IDs from the created identity:  
```
PRINCIPAL_ID=$(az identity show -n "my-container" -g "my-environment" --query principalId -o tsv)
IDENTITY_ID=$(az identity show -n "my-container" -g "my-environment" --query id -o tsv)
CLIENT_ID=$(az identity show -n "my-container" -g "my-environment" --query clientId -o tsv)
SUBSCRIPTION_ID=$(az account show --query id -o tsv)
```

Assign a role to the identity:  
```
az role assignment create --assignee $PRINCIPAL_ID  \
--role "role ID" \
--scope "id of the resource"
```

You can find the resource ID by running the `az <resource type> show -g "resource-group" -n "resource name" --query id -o tsv` command.
Role IDs take a bit more digging to find, by you can run `az role definition list` to view all roles, and do querying such as   
```bash
az role definition list --query "[?contains(roleName, 'Key Vault Reader')].{Name:name, RoleName:roleName, Permissions:permissions[].actions}" -o jsonc
```

> Read more about [JMESPath (Json Path Query languge) here](https://learn.microsoft.com/en-us/cli/azure/query-azure-cli?tabs=concepts%2Cbash).

## Creating an environment and container app
Create an environment the container app can live in.

```bash
az containerapp env create -n $ENVIRONMENT_NAME -g $RG --location westeurope
```
Then provision a container.

```bash
az containerapp create -n my-container -g my-environment \
--image $IMAGE_NAME \
--registry-server $ACR_SERVER \
--registry-user 00000000-0000-0000-0000-000000000000 \
--registry-password $(az acr login -n $ACR_NAME -g $RG --expose-token --query accessToken -o tsv) \
--environment $RG \
--ingress external --target-port 80 \
--enable-dapr \
--dapr-app-id my-container \
--dapr-app-port 80 \
--env-vars 'some=thing' \
--query properties.configuration.ingress.fqdn \
```

## Creating a postgres for containers to use
Let's create a postgres database that the containerapp can connect to.

Create the postgres server:  
Note, if the postgres server requires TLS, you'll have to download a certificate and make your app use that one.
The easiest is to disable "enforce TLS".
```bash
az postgres server create --name $POSTGRES_NAME -g $RG \
--location westeurope \
--admin-user postgres \
--admin-password postgres \
--sku-name B_Gen5_1 \
--ssl-enforcement Disabled \
--version 11
```

Create service connection:  
```bash
az containerapp connection create postgres -g $RG -n $CONTAINER_NAME -c $CONTAINER_NAME \
--target-resource-group $RG \
--server $POSTGRES_SERVER \
--database $POSTGRES_DATABASE_NAME \
--connection $YOUR_CONNECTION_NAME \
--client-type dotnet
```
This creates a firewall rule on the postgres server.
Specifying the `--client-type` creates a connection string that can be injected into the container as an environment variable.

Alternatively to creating a service connection, update the postgres server firewall to allow traffic from the container environment:  
1. Get the static IP of the container environment:  
```bash
ENV_IP=$(az containerapp env show -g $RG -n $CONTAINER_ENVIRONMENT_NAME --query properties.staticIp -o tsv)            
```

2. Update firewall rule:  
```bash
az postgres server firewall-rule create -g $RG \
--server $POSTGRES_NAME \
--name AllowIps \
--start-ip-address $ENV_IP \
--end-ip-address $ENV_IP
```

## Connecting to a keyvault
1. Create the keyvault using the following command
```bash
az keyvault create -g $RG \
-n $KEYVAULT_NAME \
--sku Standard \
--enable-rbac-authorization true
--location $LOCATION
```

2. Add reader role to a managed identity that is assigned to the containerapp
```bash
IDENTITY_ID=$(az identity show -g $RG -n $IDENTITY_NAME --query principalId -o tsv)
AZ_KEYVAULT_ID=$(az keyvault show -g $RG -n $KEYVAULT_NAME --query id -o tsv)
KV_READER_ROLE=21090545-7ca7-4776-b22c-e363652d74d2

az role assignment create --assignee-object-id $IDENTITY_ID \
--role $KV_READER_ROLE \
--scope $AZ_KEYVAULT_ID \
--assignee-principal-type ServicePrincipal
```

3. Add the keyvault as a dapr secret store component
Run `az identity show -g $RG -n $IDENTITY_NAME -o jsonc` to get all the required values for the yaml component definition.
```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: secrets
spec:
  type: secretstores.azure.keyvault
  version: v1
  metadata:
  - name: vaultName # Required
    value: nicmenvkeyvault
  - name: azureEnvironment
    value: "AZUREPUBLICCLOUD"
  - name: azureClientId
    value: <client id>
```

Read more about the azure containerapp dapr schema here: https://learn.microsoft.com/en-us/azure/container-apps/dapr-overview?tabs=bicep1%2Cyaml#component-schema

## Side notes
Solving the resource service connection for a new containerapp.
The issue is, to create a service connection, you'll need both the container and the resource, e.g. a postgres database.  
To start with you'll have e.g. a postgress database, but there's no containerapp to make the service connection to.

And, if you provision containerapp before the service connection exists, the app provisioning will fail if the app does database queries upon startup.

To solve this, you will have to
1. create a containerapp with no specified image. The containerapp will then start up using a standard "hello world" microsoft image.
2. Create the service connection
3. Run the `up` command specifying your own image and use the secret generated from the service connection.


## Useful commands
- Get a containers endpoint: `az containerapp show -g "resourge-group" -n "app name" --query properties.configuration.ingress.fqdn -o tsv`
- Add environment variable to existing revision (will perform restart): `az containerapp update -n $CONAINER_NAME -g $RG --set-env-vars 'some__value=whatever'`


## Links
- [Communication between microservices in Azure Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/communicate-between-microservices?tabs=bash&pivots=acr-remote)
- [Build and deploy a Python web app with Azure Container Apps and PostgreSQL](https://learn.microsoft.com/en-us/azure/developer/python/tutorial-deploy-python-web-app-azure-container-apps-02?tabs=azure-cli%2Ccreate-database-psql)
- [Deploy a Dapr application to Azure Container Apps using the Azure CLI](https://learn.microsoft.com/en-us/azure/container-apps/microservices-dapr?tabs=bash)
- [Dapr for .NET Developers, MSDN](https://learn.microsoft.com/en-us/dotnet/architecture/dapr-for-net-developers)
- [Dapr integration with Azure Container Apps, MSDN](https://learn.microsoft.com/en-us/azure/container-apps/dapr-overview?tabs=bicep1%2Cyaml)
- https://gist.github.com/MauricioMoraes/87d76577babd4e084cba70f63c04b07d
- https://github.com/MicrosoftDocs/azure-dev-docs/blob/main/articles/java/spring-framework/migrate-postgresql-to-passwordless-connection.md

## Tutorial - a refresher
Create resouce group  
`az group create -g my-environment --location westeurope`

Setup environment  
`az containerapp env create -n "my-containerapp-env" -g "my-environment" --location westeurope`

Create a managed identity  
`az identity create --resource-group "my-environment" --name "containerIdentity"`

Create containerapp  
```bash
az containerapp create -n my-container -g my-environment \
            --image mcr.microsoft.com/azuredocs/containerapps-helloworld:latest \
            --registry-server nicmshared.azurecr.io \
            --registry-user 00000000-0000-0000-0000-000000000000 \
            --registry-password $(az acr login -n $ACR_NAME -g $RG --expose-token --query accessToken) \
            --environment my-containerapp-env \
            --ingress external --target-port 80 \
            --user-assigned $IDENTITY_ID \
            --enable-dapr \
            --dapr-app-id my-container \
            --dapr-app-port 80 \
            --env-vars 'some=thing' \
            --query properties.configuration.ingress.fqdn \
```


## Check list
- Able to create containers
- Can use dapr
- Create managed identity
- Create container registry
- Create container app environment (dapr enabled)
- Connect dapr components
- Create container app 
