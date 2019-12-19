# Getting .NET Applications ready for CD/CI
Building software should be environment-agnostic.
Scaling-out should only require to add more deployment targets
Secrets remain secret
There should be no "noise" in source control

# Continuous Integration
Merge developer code into master branch often, build automatically, run tests
Usually one or more branches should be marked as "integration branches"


# Continuous Delivery
Deploying built software automatically with minimal impact to users, developers, or deploy staff

# Things to be wary of
 - External dependencies
 - Application configuration
 - Shared library code
 - Database updates <- databases should not be updated manually
 - Web server configuration
 - Avoid server downtime when deploying
 

# Packages
May be required to lock down dependency versions in Nuget and NPM

# Version control
Do not keep a branch per environment

# Application configuration
Connection strings
URLs
API secrets / keys
App settings

Use app settings over environment variables
May be a good idea to write a custom class to get configurations

Setup table specifying all application configurations, like below

Item  |  Definition  |  Global/App  |  Has Secrets?  |  Env-specific? |
------|--------------|--------------|----------------|----------------|


# Run-Time Requirements
- Logical entry points
- Needed applications (.NET runtime, node, etc)
- Deployment environments


# Azure DevOps Specifics

## Required build steps in Azure DevOps Pipelines
1. Define where source will come from
2. Review yaml file
3. Run build


## Yaml build file
Set variables that can be used like $(this)
All set variables will become environment variables during build


## Build triggers
Continuous integration
Branch filters
Pull requests
Gated check-in (Only merge commits after successful build)


# Different builds
CI build
	Compile/test to see code will actually build correctly
Nightly build - scheduled
	Will be deployed to test environment where e.g. integration and UI tests can be done
	- Compile
	- Regression
	- Provisinging
	- Code analysis
Release build - promoted build
	Uses the artifacts produced by a previous, promoted build
	Compile
	Test
	Regression
	Config management
	Archive

Build artifacts like zips, bin etc. should be stored in an artifact location
The artifact can be candidate for release and should be pulled by another build pipeline


# Infrastructure as Code
Services can be provisioned using file templates. The templates are checked into VCS.
The goal is to only have running environments when needed.


# Publish to web app
Get template from Azure Web App to be used for provisioning.
The template should be copied into the artifact folder of the build

Templates (provisioning scripts) can also be stored in separate GIT repo.

# Requirements
Azure Subscription
Azure DevOps Account
	Permissions to create pipelines and start builds
	Authorized Connection Service to an Azure Resource Manager
Different Environments Configured (e.g. Dev, Test, Prod)

