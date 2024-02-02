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

## Approach
- I have used separate repos for Infra and Application to fecilitate separate CI/CD for both
- Infra Repo url:
- Application Repo url:

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

  ### Azure Storage account and Blob creation
  - Search "Storage Account" -> "Create" -> Fill in the details -> "Reveiw and Create" (Enable Blob Versioning under Data Protection tab)
  - Go to the newly created Storage Account -> "Containers" under "Data Storage" -> "+ Container" -> Fill in details and "Create"
  - Note down the "Resource Group", "Storage Account", and "Container" names which are used in the "backend.tf" file on the Infra Repo

  ### Azure VM Creation
  - Search for "Azure Virtual machine" -> "Create New" -> "Fill in details" (Use a linux image) -> Review+Create
  - Ensure to enable SSH access
 
  ### Create Service Connection on Azure DevOps
  - Go to project settings -> Service Connections -> Create new -> Selecte "Azure Resource Manager"
  - Select the details partainting to your account (Client ID, Tenant ID, Client Secret, Subscription ID)
  - Create
    
  ### Add the Azure VM (or vm of choice) as agent on Azure DevOps
  - Project Settings -> Agents -> Agent Pools -> Create new -> Type "Self Hosted"
  - Open the newly created Agent Pool
  - Add New Agent
  - SSH into your VM and follow the guides given on the Azure DevOps page
  - I have named my pool "selfAgents" and added that as "pool" on the pipeline Jobs
 
  ## CI Pipeline
  - Application Repo:
      - Directory root contains the CI Pipeline YAML which used for the pipeline
      - Directory root contains the manifest for the storage class creation which is common among both deployments
      - ./mwiki-db-chart and ./mwiki-app-chart are the Helm Charts used to deploy the db and the app components/k8s objects
      - In this pipeline I have basically run few validations, tool installations and packaging and publishing
          - Install Helm, Kubectl
          - Create a dry run for the Storage Class Deployment to verify on the console

```bash
  kubectl create <manifest.yaml> --dry-run -o yaml
```
