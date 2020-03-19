# Azure Fundamentals

## Cloud concepts
When using cloud services, you are basically renting compute power from someone else.

The concepts to know about are:
- High availability
- Scalability
- Elasticity
- Agility
- Fault tolerance
- Disaster recovery
- Global reach
- Customer latency capabilites
- Predictive cost considerations
- Security

## Region

A region is an Azure data center in a specific geographic location.  
All resources are created in a subscription and region.

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

Enables you to create private networks in the cloud, giving full control over IP addresses, DNS servers, security rules, and traffic flows.

Logically isolated network on Azure. Allows secure communication between Azure resources on the network.  
Virtual networks are scoped to a single region, but can be connected with other regions thru network peering.

Virtual networks can be divided into subnets.

On-premise devices can be connected to the Azure virtual network using a VPN/virtual network gateway.

### Azure VPN Gateway

Virtual network gateway that is used to send encrypted traffic between an Azure Virtual Network and an on-premise location using the public internet.  
VNet can also be used to send encrypted traffic between Azure virtual networks over the Microsoft network.

Each virtual network can only have one VPN gateway.

### Azure ExpressRoute

Create private connections between Azure data centers and infrastructure on your environment. This does not use the public internet.

### Network security group

An NSG allows or denies inbound network traffic to the Azure resources, i.e. filtering traffic - it's like a cloud-level firewall.

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

### Inbound protection (network perimeter)

In Azure, there are three typical ways of providing inbound protection.

1. Azure Firewall
   Only specified IP addresses are allowed to make requests to the service
2. Azure Application Gateway
   Designed to protect HTTP traffic
3. Network Virtual appliances (NVAs)
   Ideal for non-HTTP services - is similar to hardware firewall appliances

### Role-based access control (RBAC)
Roles are applied to a scope where the permissions are effective.  
Scopes are:
- Management groups: used to organnize multiple subscriptions
- Subscripts
- Resouce groups
- Resources

Roles are assigned to:
- Users
- Groups
- Service principals: e.g. a resource that needs access to another resource

Roles are assigned using the Access control (IAM) menu option on e.g. a resource group or resource.  

## IT compliance

### Policies

Creating a policy starts with creating a policy definition. This has 1) conditions under which it is enforced and accompanying effect that takes place if the conditions are met.  

Multiple policies can be grouped into _initiatives_. Managing access, policies and compliance across subscriptions can be done using Azure Management Groups.  

Resources are scanned hourly for compliance with policies.  

Use Azure Policies and assignments to enforce conventions, such as naming.  

Policies can be created with the Portal, PowerShell or Azure CLI.  
To create policies, the logged in account must have the permission Microsoft.Authorization/policyassignment/write.  

#### Policy Effects
| Effect            |                                  Description                                  |
|-------------------|:-----------------------------------------------------------------------------:|
| Append            |                          Resource property additions                          |
| Audit             |                       Logging only; generates a warning                       |
| AuditIfNotExists  |                Auditing is enabled if resource does not exist                 |
| Deny              | Existing non-compliant resources are marked as non-compliant, but not deleted |
| DeployIfNotExists |               If the resource does not already exist, deploy it               |

## Organizing resources

Create resource groups to hold resources, and set up RBAC (role-based access control), to only allow access to what is needed.
Deployment - every resource is one deployment unit.

Use resource groups to organize resources. Find and use a consistent naming and grouping convention.  
For project work, it makes sense to create a resource group per project for easier billing management.

### Tags

Alternatively, tags can be used to group resources. Tags are basically meta-information attached to resources.

Place _locks_ on resources to avoid deletion by mistake. Locks can easily be removed again.

## Cost

### CapEx vs OpEx
Capital Expenditure (CapEx) is spending money on physical infrastructure upfront, and then deducting taxes over time.  
Operational Expenditure (OpEx) is pending money on services or products and being billed immediately.

Factors that affect costs are:

1. Resource type
2. Services
3. Location
4. Azure billing zones

Use Azure Pricing Calculator[1] to easily get an estimate of what services cost.
[1]: https://azure.microsoft.com/en-us/pricing/calculator/

Post-deployment, you can use the Azure Advisor to get recommendations on running services.

Azure Cost Management is another service you can use to get insights into your cloud spendings.

### Zones

A zone is a geographical grouping of Azure regions used to determine billing based on data transfers. Billing applies to both inbound and outbound traffic.  
Data transfer between zones and regions within zones are billed. Transfers between zones in the same region are not billed.

### Migrating to PaaS or SaaS services

Moving from on-premise to the cloud should be done iteratively.  
You can start by provisioning IaaS and move those to PaaS or SaaS.

## License costs

Where you have a choice and your application is indifferent to the OS, it's a good idead to choose the cheapest option.

### Existing licenses

If you have existing licenses, such as for SQL Server og Windows, you may be eligible to get a reduced rate at Azure for those services. To use an existing licens, search for BYOL (Bring Your Own License) services.  
You may also youse Dev/Test subscriptions to avoid paying licensing fees. These must only be used for development and test environments. Also, all users must be covered under a Visual Studio subscription.

## Azure Services

### Azure Security Center

### Azure Service Health

This service is composed of Azure status, the service health service, and Resource Health.  
Notifies one about current and upcoming issues such as service impacting events, planned maintenance, and other changes that affect availability.

Azure status: informs you about service outages in Azure.
Azure service health: provides information about the health of Azure services and regions you're using.
Azure resource health: informationa about the individual resources you are using.

### Azure Advisor
The advior provides recommendations for High Availablity, Security, Performance, and Cost.

### Azure Monitor

Maximizes availability and performance. Delivers a comprehensive solution for collecting, analyzing, and acting on telemetry from cloud and on-premise environments.

All data collected by the monitor is either in the form of metrics or logs.

Azure monitor can use autoscale to add or remove resources to minimize costs and ensure performance. Rules can be set up based on metrics.