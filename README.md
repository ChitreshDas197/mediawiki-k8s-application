# mediawiki-k8s-application
- This repo talks about deplopying mediawiki application on a kubernetes cluster
- I have used Azure cloud as preferred infra platform
- The infra (AKS Cluster, Azure Resource Group etc) has been provisioned using Terraform IaC
- Database (MySQL) and Web Server App (Mediawiki) has been deployed to Kubernetes using Helm Charts
- A common Storage Class has been used to dynamically provision Persistent Volume backed by Azure Files
- This Persistent Volume is consumed by each deployment (MySQL and Mediawiki) via respective Persistent Volume Claims
- Both Infra and Application has been deployed using CI/CD on Azure DevOps Pipeline
- CI Pipeline - Yaml Based
- CD/Release Pipeline - GUI Based Development (As of now Azure Pipelines does not support Release Pipelines with YAML)
- Alternatively you can create the CD within the Pipeline section as an additional Job maybe named 'Deploy'
- I have used separate release pipeline to include approval gates and conditions easily

## Application Architecture on AKS Cluster


![mediawiki-k8s-architecture](https://github.com/ChitreshDas197/mediawiki-k8s-application/assets/65863286/d4ec4838-939c-4c25-b18c-07d81452c3e0)

## Pre-requisites
- Azure Subscription and Account (Can use Free Trial as well - I have used the same)
- Azure Service Principal created
- Azure storage account and container - for terraform backend
- A linux machine (VM/Physical) hosted on any cloud/virtualization solution/on-prem - as build agent (I have used Azure VM as Self Hosted Build Agent - Can use one on Azure DevOps but currently it does not work with Free Tier accounts)

  ### Azure Service Principal Creation Steps
  - Log in to Azure Portal
  - Open Microsoft Entra ID (the new Azure Active Directory)
  - Go to "App Registration" on left blade and open
  - Hit "New Registration" and fill in details -> "Register"
  - Note down the "client ID" and "tenant ID" from the Overviews page of this newly registered app
  - Click on "Add a certificate or secret" from "Client Credentials" field -> Create a new secret and node the Client Secret from the value field somewhere
  - Go to "Subscription" -> Choose your subscipiton -> Note down the subsciption ID
  - Under "Access Control (IAM)" on the lefr blade -> "Add" -> "Add Role Assignment"
  - Under "Role" -> "Privileges Administrator Role" tab -> Choose "Contributor"
  - Under "Members" -> "Assign Access to" -> "User, group or service principal" -> Select the Registered app -> Hit "Ok"
  
