# Terraform Infrastructure Automation with Azure DevOps CI/CD

This repository demonstrates a complete end-to-end **Infrastructure as Code (IaC)** implementation using **Terraform** on **Microsoft Azure**, integrated with **Azure DevOps CI/CD pipelines** and managed through **GitHub**.  
The project follows real-world DevOps best practices such as remote backend state management, CI/CD separation, RBAC security, and approval-based deployments.

---

## 🎯 Project Objective
- Provision Azure infrastructure using Terraform  
- Avoid manual resource creation via Azure Portal  
- Store infrastructure as code in GitHub  
- Implement CI/CD pipelines using Azure DevOps  
- Use Azure Storage as a remote backend for Terraform state  
- Follow enterprise-level DevOps standards  

---

## 🛠 Tools & Technologies
- Terraform  
- Microsoft Azure  
- Azure DevOps  
- Azure Storage Account (Remote Backend)  
- Azure CLI  
- GitHub  
- Visual Studio Code (VS Code)  

---

## 📂 Repository Structure

tf_code/ │-- main.tf │-- variables.tf │-- .gitignore │-- README.md


---

## ⚙️ Terraform Configuration

**variables.tf**
```hcl
variable "location" {
  description = "Azure region"
  type        = string
  default     = "Central India"
}

variable "prefix" {
  description = "Resource naming prefix"
  type        = string
  default     = "free"
}

main.tf

terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
  skip_provider_registration = true
}

resource "azurerm_resource_group" "rg" {
  name     = "${var.prefix}-rg"
  location = var.location
}

resource "azurerm_virtual_network" "vnet" {
  name                = "${var.prefix}-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = var.location
  resource_group_name = azurerm_resource_group.rg.name
}

resource "azurerm_subnet" "subnet" {
  name                 = "${var.prefix}-subnet"
  resource_group_name  = azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.0.1.0/24"]
}

🚫 .gitignore

.terraform/
.terraform.lock.hcl
terraform.tfstate*
*.tfvars
secrets.txt
.env
*.log

💻 Local Execution (VS Code)
az login
az account set --subscription <SUBSCRIPTION_ID>

terraform init
terraform validate
terraform plan
terraform apply
terraform destroy


- terraform plan → Preview only (no cost)
- terraform apply → Creates resources
- terraform destroy → Deletes resources

📦 Remote Backend (State Management)
Terraform state is stored in an Azure Storage Account instead of locally.
Benefits:
- Centralized state management
- Team collaboration
- CI/CD pipeline compatibility
- Prevention of state conflicts

🔄 CI/CD Pipeline Design (Azure DevOps)
CI (Continuous Integration):
- Triggered on every push to main branch
- Installs Terraform
- Runs terraform init, validate, fmt -check, plan
CD (Continuous Deployment):
- Runs only if CI succeeds
- Requires manual approval
- Executes terraform apply
- Uses the same remote backend

📜 Azure DevOps Pipeline YAML
trigger:
- main

stages:
- stage: CI
  displayName: Terraform CI
  jobs:
  - job: Validate
    displayName: Validate and Plan Terraform
    pool:
      vmImage: ubuntu-latest
    steps:
    - task: TerraformInstaller@1
      displayName: Install Terraform
      inputs:
        terraformVersion: 'latest'
    - task: TerraformTask@5
      displayName: Terraform Init
      inputs:
        provider: 'azurerm'
        command: 'init'
        backendServiceArm: 'Azure-Service-Connection'
        backendAzureRmResourceGroupName: 'free-rg'
        backendAzureRmStorageAccountName: 'harshaduppalapati'
        backendAzureRmContainerName: 'prod-tfstate'
        backendAzureRmKey: 'prod.terraform.tfstate'
    - task: TerraformTask@5
      displayName: Terraform Validate
      inputs:
        provider: 'azurerm'
        command: 'validate'
    - task: TerraformTask@5
      displayName: Terraform Format Check
      inputs:
        provider: 'azurerm'
        command: 'custom'
        customCommand: 'fmt -check'
    - task: TerraformTask@5
      displayName: Terraform Plan
      inputs:
        provider: 'azurerm'
        command: 'plan'
        environmentServiceNameAzureRM: 'Azure-Service-Connection'

- stage: CD
  displayName: Terraform CD
  dependsOn: CI
  condition: succeeded()
  jobs:
  - deployment: Apply
    displayName: Terraform Apply
    environment: production
    pool:
      vmImage: ubuntu-latest
    strategy:
      runOnce:
        deploy:
          steps:
          - task: TerraformInstaller@1
            displayName: Install Terraform
            inputs:
              terraformVersion: 'latest'
          - task: TerraformTask@5
            displayName: Terraform Init
            inputs:
              provider: 'azurerm'
              command: 'init'
              backendServiceArm: 'Azure-Service-Connection'
              backendAzureRmResourceGroupName: 'free-rg'
              backendAzureRmStorageAccountName: 'harshaduppalapati'
              backendAzureRmContainerName: 'prod-tfstate'
              backendAzureRmKey: 'prod.terraform.tfstate'
          - task: TerraformTask@5
            displayName: Terraform Apply
            inputs:
              provider: 'azurerm'
              command: 'apply'
              environmentServiceNameAzureRM: 'Azure-Service-Connection'



🔐 Security Practices
- Terraform state files excluded from GitHub
- Secrets never committed
- Azure DevOps Service Connections used for authentication
- RBAC applied for Storage backend access
- Manual approval enforced before production apply
- GitHub secret scanning protections respected

📌 Why Artifacts Are Not Used
- Terraform does not generate build outputs
- Infrastructure state is stored in a remote backend
- Artifacts are used for application pipelines, not IaC pipelines

📚 Key Learnings
- Infrastructure as Code using Terraform
- Azure remote backend configuration
- CI/CD automation for infrastructure
- RBAC and data-plane permissions
- Real-world DevOps troubleshooting
- Secure cloud automation

📝 Interview Summary
"I automated Azure infrastructure provisioning using Terraform and implemented CI/CD pipelines in Azure DevOps with remote state management and approval-based deployments."


👤 Author
Harshad Krishna Uppalapati
Aspiring Azure Cloud / DevOps Engineer
