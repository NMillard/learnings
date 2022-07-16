# Create Azure App Service

HTTP-based service for hosting web apps.  
Makes it easy to run applications in different frameworks. Deploy directly or in containers.  
Security, load balancing, and automation is supported. Auto-scale at the compute layer.

Costs are determined by the App Service Plan.

## Understanding App Service Plans (ASP)
Two types: non-isolated, and isolated.  

An ASP can be one or more virtual machines behind the scene. Microsoft manages the compute layer, making sure VMs are patched, highly available, functional, and secure.

It's possible to:
- Deploy multiple app services on one ASP, and
- an ASP is scalable.

### Non-isolated
Non-isolated ASPs are _not_ inside a virtual network.  

Tiers:
- Free and Shared (F1, D1): not intended for production apps. Apps are running next to other customers' apps on the same VM. Don't use this for production.
- Basic (B1, B2, B3): low traffic requirements, don't need advanced auto-scaling. Use for robust testing.
- Standard (S1-S3): Meant for production workloads.
- Premium V2 (P1-P3v2): Enhanced performance for production apps.
- Premium V3 (P1-P3v3): Even more powerful than V2.

### Isolated
- Fully isolated and dedicated environments for running web apps.
- Designed for high scale and memory usage.
- Isolation and secure network access.
- Fine-grained control over network traffic.
- Apps can connect over VPN to on-premises resources.

App Service Environment (AES) is completely isolated and it's possible to open up for public traffic using a Virtual IP. For only-internal environments you can deploy an inside load balancer.
On premise resources can use VPN/Express route to talk to the inside load balancer.

## Creating ASPs
### Web App in the Portal
Click create resource and search for web app.
Follow the creation guide. If there's no existing ASP, then you can make one while creating the web app.

### Web App with Azure CLI
Open bash and login using `az login`.  
Create resource group `az group create -n name -l swedensouth`.  
Create ASP `az appservice plan create --name name --resource-group name --sku s1 --is-linux`  
Create web app `az webapp create -g rg-name -p asp-name -n some-name --runtime dotnet:6`.  

### Web App with PowerShell
Some approach as with az CLI.
```powershell
# Create RG
New-AzResourceGroup -Name my-rg -Location swedencentral

# Create ASP
New-AzAppServicePlan -Name my-asp -Location swedencentral -ResourceGroupName my-rg -Tier S1

# Create web app
New-AzWebApp -Name my-web-app -Location swedencentral -AppServicePlan my-asp -ResourceGroupName my-rg
```

### Web App with ARM Template
You can create ARM templates and deploy them with e.g. PowerShell, AZ CLI, or DevOps pipelines.
With AZ CLI it'd look like this:  
`az deployment group create --resource-group my-rg --template-uri http-path-to-template` or
`az deployment group create --resource-group my-rg -f path-to-file`

## Configuring Azure App Services

### Securing a web app with SSL
Map custom domain to our web apps and upload a TLS certificate for secure traffic.  

Considerations: 
- Free tiers cannot be associated with custom domain and TLS.  
- Public vs private certificates: Public certs are commonly used.
- Managed vs. unmanaged certificates: you can buy app service certs directly from Azure or upload own certificates.
- Enforce HTTPS and TLS.

To configure a web app with custom domain you'll need to create a cname and txt record with the specified values that azure shows.  
To add a TLS certificate to a web app you go into the TLS/SSL Settings and click "Buy certificate" or upload a certificate from disk, keyvault, or create a free App Service Managed certificate.  

When there's a certificate available, you need to create a TLS binding to it. There are two types:
- IP based SSL: when you have an IP Based SSL certificate.
- SNI SSL: Server Name Indication, ability to run multiple TLS certs for multiple domains on same IP address.

### Setting up database connectionstring
Setting up a connectionstring on a web app is basically just an environment variable on the compute environment.  
It's important the connectionstring is not hardcoded into the application code.  

To set the connectionstring, go to webapp -> configuration -> Application settings -> Connection strings.

### Logging
You an enable diagnostic logging in web apps.  
Application logging is specific to the framework your app is developed in.  
Web server logging: each request and response is logged on the web server. Stored on disk or storage account.  
Detailed error messages (windows): turn on when developing.
Failed request tracing (windows)  
Deployment: logs when pushing code to the service.

Turn on logging by going to App Service Logs. Logs stored on file system can be accessed by FTP.

### Deploying an app service
It's possible to do continuous integration (deployment) where every commit is automatically deployed.  
This requires Azure App Service authentication against the source repo software.  
Continuous deployment also requires a build server. The simplest option is to use the kudu integrated build server, that picks up the latest commit, builds the code and deploys it.

## Scaling Azure App Services
Vertical scaling: upping the compute power of a single app service, e.g. buy more RAM, CPU cores, etc.
Horizontal scaling: traditionally done by provisioning a load balancer and connect multiple VMs to it.
