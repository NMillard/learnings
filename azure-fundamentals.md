# Azure Fundamentals

## Subscription

This is basically a logical container used to provision resources. Every resource created will be associated with a subscription.  
Every subscription has a unique storage account associated with it.

It makes sense to create additonal subscription to separate:

1. Environments
2. Organizational structure
3. Billing

### Billing

Multiple subscriptions can be organized into invoice sections within a billing profile.

## Azure CLI

Here's a list of useful commands

Show table list of subscriptions  
`az account list --output table`

Show table list of resource groups  
`az group list --output table`

Show all resources  
`az resouce list`

Show all resources in specific group  
`az resouce list --resource-group <group id>`

## Networking

### Azure virtual network

Logically isolated network on Azure. Allows secure communication between Azure resources on the network.  
Virtual networks are scoped to a single region, but can be connected with other regions thru network peering.

Virtual networks can be divided into subnets.

On-premise devices can be connected to the Azure virtual network using a VPN/virtual network gateway.

### Network security group

An NSG allows or denies inbound network traffic to the Azure resources - it's like a cloud-level firewall.

## Security

Microsoft provides Azure components that enables you to secure your resources, but it you'll have to make use of those components yourself.

Azure uses Defence in depth, which means every layer has its own security.  
Going from Physical Security -> Identity & Access -> Perimeter -> Network -> Compute -> Application -> Data.

- Data
  - Ensure proper access controls to databases
- Application
  - Store sensitive application secrets in key vault
  - Never check in secrets in source control
  - Follow Security by design principals
- Compute
  - Secure access to VMs, App services, etc.
- Networking
  - Limit communication between resources
  - Deny by default
  - Only have required ports open
- Identity & Access

  - Use SSO and MFA
  - Avoid storing user data yourself (use AD, Facebook, Google, etc. instead)

### Identity and principals

An identity is anything that can be authenticated, for example users and applications (resources).  
A principal is an identity acting with certain roles or claims.

In Azure, if you have two resources that need to interact, you can create a managed identity for a service (resource).  
Creating a managed identity will create an account on your organization's AD.

This allows you to authenticate one resource with another thru AD, and define what the resource is allowed to do.

### Certificates

Azure uses x.509 v3 certs.  
They are used for two reasons, 1) cloud services and authenticating with the management API.  
Service certificates are attached to services to ensure secure communication to and from the service. Management certificates are for instance used by tools such as Azure SDK or Visual Studio, to authenticate with Azure to allow deployments.

Certificates can be stored in Azure Key Vault, which allows auto-renewal.

## Inbound protection (network perimeter)

In Azure, there are three typical ways of providing inbound protection.

1. Azure Firewall
   Only specified IP addresses are allowed to make requests to the service
2. Azure Application Gateway
   Designed to protect HTTP traffic
3. Network Virtual appliances (NVAs)
   Ideal for non-HTTP services - is similar to hardware firewall appliances

## IT compliance

### Policies

Creating a policy starts with creating a policy definition. This has 1) conditions under which it is enforced and accompanying effect that takes place if the conditions are met.

Multiple policies can be grouped into _initiatives_. Managing access, policies and compliance across subscriptions can be done using Azure Management Groups.

## Organizing resources

Create resource groups to hold resources, and set up RBAC (role-based access control), to only allow access to what is needed.
Deployment - every resource is one deployment unit.

Use resource groups to organize resources. Find and use a consistent naming and grouping convention.  
For project work, it makes sense to create a resource group per project for easier billing management.

Alternatively, tags can be used to group resources.

Use Azure Policies and assignments to enforce conventions.

Place _locks_ on resources to avoid deletion by mistake. Locks can easily be removed again.

## Cost

Factors that affect costs are:

1. Resource type
2. Services
3. Location
4. Azure billing zones
