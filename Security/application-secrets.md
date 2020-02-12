# ASP.NET Core Application Secrets

Any data that should not be widely known.

## Separation of Duties

An individual person or system should not have all the necessary actions to perform malicious action.

All environments and applications should have their own unique credentials.

### Database

An application should only be allowed to execute the required queries against the database, for the application
to function correctly.

### Storing Secrets

Note, if secrets have by mistake been committed to source control, you must remove and change them!

In code <- should never be done  
Configuration files <- is better than in code, but must have discipline to not check them into source control  
Environment variables <- must make sure programs does not access them  
Secrets file <- loaded at application startup

In development, use dotnet user-secrets  
Add <UserSecretsId>GUID</UserSecretsId> to the csproj file

In Production, use a key vault
Add e.g. an Azure Key Vault as secrets provider for the ASP.NET app

To authenticate with the Azure Key Vault, you need to use Azure AD authentication.
For this, you either need to 1) store one client secret, 2) use a certificate located on the web server or 3) use managed identity

A key vault should have network restriction enabled.

#### Steps

1. Add NuGet package for Azure Key Vault: Microsoft.Extensions.Configuration.AzureKeyVault
2. Add configuration build
3. Testing locally: authenticate using Azure CLI
4. Allow App Service to connect to Azure Key Vault
   1. Turn on Identity in the App Service
   2. Add the app service to Key vault's policy as principal with Get and List secret permissions
