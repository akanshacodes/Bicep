# Azure Bicep Complete Notes — End-to-End Landing Zone Setup Guide

> Beginner → Advanced | Real-Time Industry Approach | Reusability Focus | Production Best Practices

---

#  What is Bicep?

Bicep is a Domain Specific Language (DSL) developed by Microsoft for deploying Azure Infrastructure using Infrastructure as Code (IaC).

It is a modern alternative to ARM Templates.

Instead of manually creating resources from Azure Portal:
---

## 🚀 Benefits of Infrastructure as Code

- ✅ Infrastructure is fully automated
- ✅ Deployments become repeatable
- ✅ Reusable modules improve scalability
- ✅ Git provides version control
- ✅ Consistency across all environments
- ✅ Faster deployments with CI/CD
- ✅ Reduced manual errors

---

# 🧠 Why Bicep?

| Feature         | Benefit                     |
| --------------- | --------------------------- |
| Simple Syntax   | Easier than ARM JSON        |
| Reusability     | Modules can be reused       |
| Automation      | CI/CD friendly              |
| Version Control | Store in Git                |
| Standardization | Same infra everywhere       |
| Secure          | Policy + Validation support |
| Fast Deployment | Automated infra creation    |

---

# 🏗️ What is Azure Landing Zone?

Azure Landing Zone means:

👉 A pre-designed Azure environment setup using best practices.

It includes:

* Networking
* Security
* Governance
* Identity
* Monitoring
* Connectivity
* Resource Organization
* Policies
* RBAC
* Logging
* CI/CD

---

# 🎯 Goal of Landing Zone

A Landing Zone should be:

✅ Scalable
✅ Secure
✅ Reusable
✅ Automated
✅ Governed
✅ Cost Optimized
✅ Production Ready

---

# 🧱 Azure Landing Zone Architecture

```text
Management Group
    ↓
Subscription
    ↓
Resource Groups
    ↓
Networking Layer
    ↓
Security Layer
    ↓
Shared Services
    ↓
Application Layer
    ↓
Monitoring & Logging
```

---

# 📁 Recommended Folder Structure (IMPORTANT)

```text
landing-zone/
│
├── main.bicep
├── main.parameters.json
├── modules/
│   ├── resource-group.bicep
│   ├── vnet.bicep
│   ├── nsg.bicep
│   ├── storage-account.bicep
│   ├── keyvault.bicep
│   ├── vm.bicep
│   ├── aks.bicep
│   ├── appgateway.bicep
│   ├── loganalytics.bicep
│   └── monitoring.bicep
│
├── environments/
│   ├── dev.parameters.json
│   ├── qa.parameters.json
│   ├── prod.parameters.json
│
├── scripts/
│   ├── deploy.ps1
│   └── cleanup.ps1
│
└── pipeline/
    └── azure-pipelines.yml
```

---

# 🔥 Why Modular Architecture?

Without modules:

❌ Huge messy file
❌ Difficult maintenance
❌ Duplicate code
❌ Hard troubleshooting

With modules:

✅ Reusable
✅ Clean structure
✅ Easy troubleshooting
✅ Standardized infra
✅ Team collaboration easier

---

# 📦 Bicep Basics

---

# 1️⃣ Parameters

Used for user input.

```bicep
param location string = 'Central India'
```

---

# 2️⃣ Variables

Reusable internal values.

```bicep
var environment = 'dev'
```

---

# 3️⃣ Resources

Azure resources creation.

```bicep
resource rg 'Microsoft.Resources/resourceGroups@2022-09-01' = {
  name: 'dev-rg'
  location: location
}
```

---

# 4️⃣ Outputs

Display deployment values.

```bicep
output rgName string = rg.name
```

---

# 5️⃣ Decorators

Validation & metadata.

```bicep
@minLength(3)
@maxLength(24)
param storageName string
```

---

# 🚀 Resource Group Module

## modules/resource-group.bicep

```bicep
param rgName string
param location string

resource rg 'Microsoft.Resources/resourceGroups@2022-09-01' = {
  name: rgName
  location: location
}

output resourceGroupName string = rg.name
```

---

# 🌐 Virtual Network Module

## modules/vnet.bicep

```bicep
param vnetName string
param location string
param addressPrefix string

resource vnet 'Microsoft.Network/virtualNetworks@2023-02-01' = {
  name: vnetName
  location: location

  properties: {
    addressSpace: {
      addressPrefixes: [
        addressPrefix
      ]
    }
  }
}
```

---

# 🔐 NSG Module

## modules/nsg.bicep

```bicep
param nsgName string
param location string

resource nsg 'Microsoft.Network/networkSecurityGroups@2023-02-01' = {
  name: nsgName
  location: location
}
```

---

# 💾 Storage Account Module

```bicep
param storageName string
param location string

resource stg 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'

  properties: {
    allowBlobPublicAccess: false
    minimumTlsVersion: 'TLS1_2'
  }
}
```

---

# 🔑 Key Vault Module

```bicep
param keyVaultName string
param location string
param tenantId string

resource kv 'Microsoft.KeyVault/vaults@2023-02-01' = {
  name: keyVaultName
  location: location

  properties: {
    tenantId: tenantId
    sku: {
      family: 'A'
      name: 'standard'
    }
    enableRbacAuthorization: true
  }
}
```

---

# ☸️ AKS Module

```bicep
param aksName string
param location string

resource aks 'Microsoft.ContainerService/managedClusters@2023-01-01' = {
  name: aksName
  location: location

  properties: {
    dnsPrefix: aksName

    agentPoolProfiles: [
      {
        name: 'nodepool1'
        count: 2
        vmSize: 'Standard_DS2_v2'
        mode: 'System'
      }
    ]

    identity: {
      type: 'SystemAssigned'
    }
  }
}
```

---

# 🛡️ Application Gateway + WAF

```bicep
resource appgw 'Microsoft.Network/applicationGateways@2023-02-01' = {
  name: 'appgw-prod'
  location: location

  properties: {
    sku: {
      name: 'WAF_v2'
      tier: 'WAF_v2'
    }
  }
}
```

---

# 📊 Log Analytics Workspace

```bicep
resource logws 'Microsoft.OperationalInsights/workspaces@2022-10-01' = {
  name: 'log-workspace'
  location: location

  properties: {
    sku: {
      name: 'PerGB2018'
    }
  }
}
```

---

# 🧩 Main File (Orchestration Layer)

## main.bicep

```bicep
param location string = 'Central India'

module rg './modules/resource-group.bicep' = {
  name: 'resourceGroupDeployment'
  params: {
    rgName: 'prod-rg'
    location: location
  }
}

module vnet './modules/vnet.bicep' = {
  name: 'vnetDeployment'
  params: {
    vnetName: 'prod-vnet'
    location: location
    addressPrefix: '10.0.0.0/16'
  }
}
```

---

# 🌍 Multi-Environment Design

## Why Important?

Production environments should never share same values.

Different:

* VM Sizes
* Naming
* Networking
* Secrets
* Scaling
* Policies

---

# 📁 Environment Parameter Files

## dev.parameters.json

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "value": "Central India"
    }
  }
}
```

---

# 🚀 Deployment Commands

## Login

```bash
az login
```

---

# Create Resource Group

```bash
az group create \
--name prod-rg \
--location centralindia
```

---

# Deploy Bicep

```bash
az deployment group create \
--resource-group prod-rg \
--template-file main.bicep \
--parameters @dev.parameters.json
```

---

# 🔥 Best Practices (VERY IMPORTANT)

---

# 1️⃣ Use Modular Design

✅ Small reusable modules
✅ Single responsibility per module

---

# 2️⃣ Never Hardcode Values

❌ Bad:

```bicep
location: 'Central India'
```

✅ Good:

```bicep
location: location
```

---

# 3️⃣ Use Naming Standards

Example:

```text
<env>-<resource>-<region>
```

Example:

```text
prod-vnet-ci
qa-aks-ci
```

---

# 4️⃣ Secure Everything

✅ Private Endpoints
✅ Key Vault
✅ RBAC
✅ NSGs
✅ TLS1_2
✅ WAF

---

# 5️⃣ Use Tags

```bicep
tags: {
  Environment: 'Production'
  Owner: 'DevOps'
}
```

---

# 6️⃣ Environment Isolation

Separate:

* Dev
* QA
* UAT
* Prod

Never mix production resources.

---

# 7️⃣ Use CI/CD

Never deploy manually in production.

Use:

✅ Azure DevOps
✅ GitHub Actions

---

# 🔄 CI/CD Pipeline for Bicep

## azure-pipelines.yml

```yaml
trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:
- task: AzureCLI@2
  inputs:
    azureSubscription: 'Azure-Service-Connection'
    scriptType: bash
    scriptLocation: inlineScript
    inlineScript: |
      az deployment group create \
      --resource-group prod-rg \
      --template-file main.bicep
```

---

# 🔐 DevSecOps Integration

---

# Checkov Scan

```yaml
- script: checkov -d .
```

---

# TFSec Scan

```yaml
- script: tfsec .
```

---

# Secret Scan

```yaml
- script: trufflehog filesystem .
```

---

# 🛡️ Enterprise Security Best Practices

| Area              | Best Practice    |
| ----------------- | ---------------- |
| Networking        | Private Subnets  |
| Secrets           | Key Vault        |
| Access            | RBAC             |
| Internet Exposure | App Gateway WAF  |
| Monitoring        | Azure Monitor    |
| Logging           | Log Analytics    |
| Data Security     | Private Endpoint |
| Identity          | Managed Identity |

---

# ☁️ Recommended Landing Zone Components

| Layer      | Services              |
| ---------- | --------------------- |
| Identity   | Azure AD              |
| Governance | Policy + RBAC         |
| Networking | VNet + NSG + Firewall |
| Security   | WAF + Defender        |
| Monitoring | Log Analytics         |
| Compute    | AKS / VMSS            |
| Storage    | Storage Account       |
| Secrets    | Key Vault             |

---

# 🧠 Real-Time Production Flow

```text
Developer Pushes Code
        ↓
Pull Request Raised
        ↓
Validation Pipeline Triggered
        ↓
Security Scan
        ↓
Bicep Validation
        ↓
Approval
        ↓
Deployment Pipeline
        ↓
Azure Infrastructure Created
        ↓
Monitoring + Alerts Enabled
```

---

# 🔥 Advanced Bicep Concepts

---

# Conditional Deployment

```bicep
param deployStorage bool = true

resource stg 'Microsoft.Storage/storageAccounts@2023-01-01' = if (deployStorage) {
  name: 'prodstorage123'
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}
```

---

# Loops in Bicep

```bicep
param storageAccounts array = [
  'stg1'
  'stg2'
]

resource stg 'Microsoft.Storage/storageAccounts@2023-01-01' = [for name in storageAccounts: {
  name: name
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}]
```

---

# Existing Resource Reference

```bicep
resource existingVnet 'Microsoft.Network/virtualNetworks@2023-02-01' existing = {
  name: 'shared-vnet'
}
```

---

# 📊 Monitoring Strategy

Must monitor:

✅ CPU
✅ Memory
✅ Pod Restart
✅ Disk Usage
✅ Failed Login
✅ Network Traffic

---

# 📌 Azure Services You Must Learn for Landing Zone

## Networking

* VNet
* Subnet
* NSG
* Route Table
* Private Endpoint
* Application Gateway
* Firewall

---

## Security

* RBAC
* Managed Identity
* Key Vault
* Defender for Cloud
* Policies

---

## Monitoring

* Azure Monitor
* Log Analytics
* KQL
* Alerts

---

## Compute

* VM
* VMSS
* AKS

---

# 🎯 Final Goal

After learning this properly:

✅ You should be able to:

* Create reusable infrastructure
* Build enterprise landing zones
* Automate Azure infra
* Implement security best practices
* Write production-level Bicep
* Integrate CI/CD
* Deploy AKS infrastructure
* Handle real projects

---

# 🏁 Final Interview Line

> “I use Bicep to build reusable, modular, secure, and production-ready Azure Landing Zone infrastructure following Infrastructure as Code and DevOps best practices.”

---

# 📚 NEXT LEARNING ROADMAP

## Phase 1

✅ Bicep Basics
✅ Modules
✅ Parameters
✅ Variables

---

## Phase 2

✅ Networking
✅ NSG
✅ Storage
✅ Key Vault

---

## Phase 3

✅ AKS
✅ App Gateway
✅ Monitoring
✅ Private Endpoints

---

## Phase 4

✅ CI/CD
✅ DevSecOps
✅ GitOps
✅ Production Deployments

---

# 🔍 DEEP UNDERSTANDING — WHY EACH THING IS USED

---

# 📌 Why Resource Groups?

Resource Group is a logical container in Azure.

Inside Resource Group:

* VMs
* Storage Accounts
* VNets
* AKS
* Key Vault

all resources are grouped together.

## Why Important?

✅ Easy management
✅ Easy deletion
✅ Access control
✅ Billing separation
✅ Environment isolation

---

# 📌 Why Virtual Network (VNet)?

VNet provides private networking in Azure.

It works like your own private data center network.

Inside VNet:

* Subnets
* VMs
* AKS
* Databases

communicate securely.

## Why Important?

✅ Secure communication
✅ Network isolation
✅ Internal communication
✅ Private architecture

---

# 📌 Why Subnets?

Subnets divide VNet into smaller networks.

Example:

| Subnet     | Purpose          |
| ---------- | ---------------- |
| web-subnet | Frontend servers |
| app-subnet | Backend apps     |
| db-subnet  | Databases        |

## Best Practice

Always separate:

* Web Layer
* App Layer
* Database Layer

---

# 📌 Why NSG?

NSG = Network Security Group.

Works like firewall.

Controls:

* Inbound traffic
* Outbound traffic

## Example

Allow:

* Port 80
* Port 443

Deny:

* Public RDP
* Public SSH

---

# 📌 Why ASG?

ASG = Application Security Group.

Instead of managing rules using IPs:

We group resources logically.

Example:

* WEB-ASG
* APP-ASG
* DB-ASG

## Benefit

✅ Easier NSG management
✅ Better readability
✅ Cleaner architecture

---

# 📌 Why Key Vault?

Key Vault stores:

* Passwords
* Secrets
* Certificates
* Connection Strings

## Why Important?

❌ Never hardcode secrets in code.

✅ Store securely in Key Vault.

---

# 📌 Why Private Endpoint?

Private Endpoint provides private connectivity.

Without Private Endpoint:

❌ Database accessible publicly.

With Private Endpoint:

✅ Traffic stays inside Azure network.

## Best Practice

Use Private Endpoint for:

* SQL
* Storage
* Key Vault

---

# 📌 Why Application Gateway?

Application Gateway works as:

* Load Balancer
* Reverse Proxy
* SSL Terminator
* Web Traffic Controller

## Important Features

✅ Path-based routing
✅ SSL termination
✅ WAF support
✅ Layer 7 routing

---

# 📌 Why WAF?

WAF = Web Application Firewall.

Protects application from:

* SQL Injection
* XSS
* OWASP attacks

## Best Practice

Production apps exposed to internet should use WAF.

---

# 📌 Why AKS?

AKS = Azure Kubernetes Service.

Used for:

* Container orchestration
* Auto scaling
* Microservices
* High availability

---

# 📌 Why ACR?

ACR = Azure Container Registry.

Stores Docker images securely.

Pipeline Flow:

Code → Docker Image → Push to ACR → Deploy to AKS

---

# 📌 Why Azure Monitor?

Azure Monitor collects:

* Metrics
* Logs
* Alerts
* Performance data

## Why Important?

Without monitoring:

❌ No visibility
❌ No troubleshooting

---

# 📌 Why Log Analytics?

Central place for logs.

Can query logs using KQL.

Example:

* Failed requests
* CPU spikes
* Pod crashes
* Security events

---

# 📌 Why Managed Identity?

Managed Identity removes need of passwords.

Instead of:

❌ Hardcoding credentials

Use:

✅ Azure-managed authentication

---

# 📌 Why RBAC?

RBAC = Role Based Access Control.

Controls who can:

* Read
* Write
* Delete
* Manage

resources.

## Best Practice

Give minimum required access.

Principle:

✅ Least Privilege Access

---

# 📌 Why Modules?

Suppose:

You create same VNet code again and again.

Without modules:

❌ Duplicate code
❌ Difficult maintenance

With modules:

✅ Reusable infra
✅ Easier management
✅ Faster deployment

---

# 📌 Why Parameter Files?

Different environments need different values.

Example:

| Environment | VM Size |
| ----------- | ------- |
| Dev         | Small   |
| QA          | Medium  |
| Prod        | Large   |

Parameter files help manage environment-specific values.

---

# 📌 Why CI/CD for Infrastructure?

Infrastructure should also follow DevOps.

Benefits:

✅ Automated deployment
✅ Version control
✅ Approval process
✅ Security scanning
✅ Faster deployment

---

# 🏗️ COMPLETE LANDING ZONE IMPLEMENTATION FLOW

---

# Step 1️⃣ Create Management Structure

Create:

* Management Groups
* Subscriptions
* Resource Groups

---

# Step 2️⃣ Configure Identity

Setup:

* Azure AD
* RBAC
* Managed Identity
* Service Principals

---

# Step 3️⃣ Build Networking Layer

Create:

* VNet
* Subnets
* NSGs
* Route Tables
* Firewall
* Private DNS

---

# Step 4️⃣ Security Layer

Setup:

* Key Vault
* WAF
* Defender for Cloud
* Policies
* Private Endpoints

---

# Step 5️⃣ Monitoring Layer

Configure:

* Azure Monitor
* Log Analytics
* Alerts
* Dashboards

---

# Step 6️⃣ Shared Services

Deploy:

* ACR
* Bastion
* Storage Accounts
* Jumpbox

---

# Step 7️⃣ Application Layer

Deploy:

* AKS
* VMSS
* App Gateway
* Databases

---

# Step 8️⃣ DevOps Layer

Configure:

* GitHub/Azure DevOps
* PR Validation
* CI/CD Pipelines
* Security Scans

---

# 🧠 REAL PRODUCTION BEST PRACTICES

| Area          | Best Practice                 |
| ------------- | ----------------------------- |
| Secrets       | Store in Key Vault            |
| Networking    | Use Private Endpoints         |
| Public Access | Disable wherever possible     |
| AKS           | Use separate node pools       |
| Storage       | Disable public blob access    |
| Monitoring    | Enable diagnostics everywhere |
| Pipelines     | Use PR validation             |
| Security      | Integrate Checkov/TFSec       |
| Identity      | Use Managed Identity          |
| Logging       | Centralized Log Analytics     |
| Governance    | Use Tags + Policies           |

---

# 📂 ENTERPRISE LEVEL RECOMMENDED MODULES

```text
modules/
│
├── networking/
│   ├── vnet.bicep
│   ├── subnet.bicep
│   ├── nsg.bicep
│   └── route-table.bicep
│
├── security/
│   ├── keyvault.bicep
│   ├── waf.bicep
│   └── defender.bicep
│
├── compute/
│   ├── vm.bicep
│   ├── vmss.bicep
│   └── aks.bicep
│
├── monitoring/
│   ├── loganalytics.bicep
│   ├── alerts.bicep
│   └── diagnostics.bicep
│
└── shared/
    ├── acr.bicep
    ├── storage.bicep
    └── bastion.bicep
```

---

# 🧪 HOW TO TEST BICEP BEFORE DEPLOYMENT

---

# Validate Syntax

```bash
az bicep build --file main.bicep
```

---

# What-if Deployment

```bash
az deployment group what-if \
--resource-group prod-rg \
--template-file main.bicep
```

## Why Important?

Shows:

* What will be created
* What will change
* What will be deleted

before actual deployment.

---

# 🚨 COMMON MISTAKES

| Mistake             | Problem                    |
| ------------------- | -------------------------- |
| Hardcoded values    | No reusability             |
| Single huge file    | Difficult maintenance      |
| No monitoring       | No troubleshooting         |
| Public database     | Security risk              |
| No tags             | Governance issues          |
| No parameterization | No environment flexibility |

---

# 🔥 FINAL REAL-WORLD ENTERPRISE FLOW

```text
Developer Pushes Code
        ↓
Feature Branch Created
        ↓
Pull Request Raised
        ↓
PR Validation Pipeline
        ↓
Checkov + TFSec + Secret Scan
        ↓
Approval
        ↓
Bicep Validation
        ↓
Deployment Pipeline
        ↓
Infrastructure Created
        ↓
Monitoring + Alerts Enabled
        ↓
Application Deployment
```

---

# 📚 RECOMMENDED LEARNING ORDER

## Beginner

✅ Resource Groups
✅ Storage Account
✅ VNet
✅ NSG

---

## Intermediate

✅ Modules
✅ Parameters
✅ Loops
✅ Conditions
✅ Outputs

---

## Advanced

✅ AKS
✅ App Gateway
✅ Private Endpoints
✅ Monitoring
✅ DevSecOps

---

## Expert Level

✅ Enterprise Landing Zones
✅ Multi-subscription Architecture
✅ Governance
✅ CI/CD Automation
✅ GitOps

---

# 🎉 Congratulations

You now have a production-style roadmap for learning Azure Bicep and building enterprise-grade Landing Zones.

This document is designed in a way that you can:

✅ Learn Bicep deeply
✅ Understand WHY resources are used
✅ Build reusable modules
✅ Deploy production-grade Azure infrastructure
✅ Implement DevOps best practices
✅ Create secure Landing Zones
✅ Automate infrastructure end-to-end
