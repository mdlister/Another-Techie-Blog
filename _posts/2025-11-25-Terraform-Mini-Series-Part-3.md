---
layout: post
comments: true
thumb: Terraform_on_Azure.jpeg
smallthumb: Terraform
title: "No More Secret Rotas: Azure DevOps to Azure with Workload Identity Federation"
tagline: Learn Terraform in a mini series for Azure and M365
description: "Configure WIF for Azure DevOps pipelines. If org policy mandates secrets, store them in Key Vault and plan rotation."
slug: terraform-mini-series-part-3
tags: [terraform, azure, azure-devops, workload-identity, oidc, key-vault, authentication]
canonical_url: https://anothertechieblog.co.uk/terraform-mini-series-part-3
modified: 2025-11-25
---

> **Series goal (reminder):** Stand up a practical, multi‑environment Terraform platform on Azure DevOps (with split pipelines for Infra/Entra/MS Graph), using secure auth, remote state, and reusable modules—scaling from *Dev* to *Prod*.

# Day 3 — No More Secret Rotas: Workload Identity Federation vs App Registration

---

## What You'll Build Today

- **WIF Deep Dive**: Understand why we struggled with WIF in Day 1 and how we fixed it
- **App Registration Fallback**: Create an App Registration with Terraform-managed secret rotation
- **30-Day Rotation Strategy**: Implement automatic monthly secret refresh using Terraform
- **Comparison Framework**: When to use WIF vs App Registration in real organisations
- **Production Patterns**: Extend this to service accounts and other App Registrations

> **Days 1-2 Recap**: We got WIF working after troubleshooting Terraform task versions, created remote state with locking, and set up Key Vault integration. Now let's understand the "why" behind our authentication choices.


---

## Why Authentication Strategy Matters

In Day 1, we hit a major roadblock: **WIF authentication failures** that took hours to troubleshoot. The root cause? Using the wrong Terraform task version and authentication parameters. This experience highlights why understanding authentication methods is crucial.

**The Problem with Secrets:**
- 🔑 **Manual rotation** - someone always forgets
- ⏰ **Pipeline failures** at 3 AM when secrets expire  
- 🔒 **Security risks** from long-lived credentials
- 📋 **Compliance overhead** for audit trails

**WIF Solution:**
- ✅ **No secrets** to rotate or manage
- ✅ **Short-lived tokens** (typically 1 hour)
- ✅ **Automatic renewal** by Azure DevOps
- ✅ **Recommended by Microsoft** for new projects

---

## Step 1 — WIF: What Went Wrong in Day 1 and Why It Matters

Remember this error from Day 1?

### This took 4 hours to troubleshoot!
```text
ERROR: Authentication failed using OIDC...
```

The Root Cause: Terraform Task Version Mismatch
In Day 1, we discovered that **TerraformTask@5** with specific parameters was essential:

Wrong approach (what failed initially):

```yaml
- task: TerraformTask@4  # ← Wrong version
  inputs:
    backendAzureRmUseEntraIdForAuthentication: true  # ← Misleading parameter
```

Correct approach (what worked):

```yaml
- task: TerraformTask@5  # ← Must be v5
  inputs:
    backendAzureRmUseEntraIdForAuthentication: false  # ← Counter-intuitive!
    backendAzureRmUseCliFlagsForAuthentication: true   # ← This enables OIDC
```

Why This Configuration Works
The backendAzureRmUseCliFlagsForAuthentication: true parameter tells the Terraform task to use the Azure CLI authentication context, which automatically picks up the OIDC token from Azure DevOps. This is the magic that makes WIF work.

WIF Architecture Recap

```text
Azure DevOps Pipeline 
    ↓ (OIDC Token)
Entra ID (Azure AD)
    ↓ (Validates Federation)
Azure Resources (ARM)
```

No secrets in the chain - just temporary tokens that Azure DevOps manages automatically.

# Step 2 — App Registration Fallback: For When WIF Isn't Possible
While WIF is ideal, some organisations can't use it due to:

Legacy compliance requirements

Third-party tool limitations

Network security policies

Existing investment in secret management

For these scenarios, let's build a robust App Registration with Terraform-managed secret rotation that automatically refreshes every 30 days.

# 2.1 Create the App Registration Module
Create /codebase/modules/azure/app-registration/main.tf

```hcl
# App Registration for Terraform pipeline (fallback option)
resource "azurerm_application" "terraform" {
  display_name = "app-terraform-${var.environment}-uks"
  owners       = [data.azurerm_client_config.current.object_id]
  
  prevent_duplicate_names = true

  tags = {
    environment = var.environment
    managed-by  = "terraform"
    purpose     = "terraform-pipeline"
  }
}

resource "azurerm_service_principal" "terraform" {
  application_id = azurerm_application.terraform.application_id
  owners         = [data.azurerm_client_config.current.object_id]

  tags = {
    environment = var.environment
    managed-by  = "terraform"
  }
}

# Client secret with 180-day maximum lifespan but 30-day rotation
resource "azurerm_application_password" "terraform" {
  application_id = azurerm_application.terraform.id
  display_name   = "secret-${var.rotation_trigger}"
  
  # 180-day maximum lifespan (Azure limit)
  end_date_relative = "4320h" # 180 days
  
  # Force rotation when the trigger changes
  rotate_when_changed = {
    rotation = var.rotation_trigger
  }
}

# RBAC Assignment for Azure resources
resource "azurerm_role_assignment" "contributor" {
  scope                = var.subscription_id
  role_definition_name = "Contributor"
  principal_id         = azurerm_service_principal.terraform.object_id
}

# Storage permissions for state management
resource "azurerm_role_assignment" "storage_contributor" {
  scope                = var.state_storage_account_id
  role_definition_name = "Storage Blob Data Contributor"
  principal_id         = azurerm_service_principal.terraform.object_id
}
```
/codebase/modules/azure/app-registration/variables.tf

```hcl
variable "environment" { type = string }
variable "subscription_id" { type = string }
variable "state_storage_account_id" { type = string }
variable "rotation_trigger" { 
  type    = string
  default = "initial"
}
```

2.2 Monthly Rotation Trigger Strategy
Add to /codebase/env/dev/main.tf

```hcl
# Monthly rotation trigger - changes value each month
locals {
  # This creates a new value each month, forcing secret rotation
  # Format: YYYY-MM (changes monthly)
  monthly_rotation_trigger = formatdate("YYYY-MM", timestamp())
  
  # Alternative: Force rotation on specific schedule
  # monthly_rotation_trigger = "rotation-${formatdate("YYYY-MM", timestamp())}"
}

# Update your core infra module call
module "core_infra" {
  source           = "../../modules/azure/core-infra"
  environment      = "dev"
  subscription_id  = var.subscription_id
  state_rg_name    = "rg-tfstate-core-uks"
  state_sa_name    = "sttfstate9683"
  state_container  = "tfstate"
  pipeline_principal_id = var.pipeline_principal_id
}

# App Registration with monthly rotation
module "app_registration" {
  source = "../../modules/azure/app-registration"
  
  environment             = "dev"
  subscription_id         = var.subscription_id
  state_storage_account_id = module.core_infra.state_storage_account_id
  rotation_trigger        = local.monthly_rotation_trigger
}
```
2.3 Store Secrets in Key Vault with Compliance Tracking
Add to /codebase/modules/azure/core-infra/main.tf

```hcl
# Store App Registration secrets in Key Vault
resource "azurerm_key_vault_secret" "terraform_client_id" {
  name         = "terraform-appreg-client-id"
  value        = module.app_registration.client_id
  key_vault_id = azurerm_key_vault.secrets.id
  
  tags = {
    secret-type   = "app-registration"
    environment   = var.environment
    managed-by    = "terraform"
  }
}

resource "azurerm_key_vault_secret" "terraform_client_secret" {
  name         = "terraform-appreg-client-secret"
  value        = module.app_registration.client_secret
  key_vault_id = azurerm_key_vault.secrets.id
  
  # Set 30-day expiration for compliance tracking
  expiration_date = timeadd(timestamp(), "720h") # 30 days
  
  tags = {
    secret-type   = "client-secret"
    environment   = var.environment
    rotation      = "monthly-terraform"
    last_rotated  = timestamp()
    managed-by    = "terraform"
  }
}

resource "azurerm_key_vault_secret" "terraform_tenant_id" {
  name         = "terraform-appreg-tenant-id"
  value        = data.azurerm_client_config.current.tenant_id
  key_vault_id = azurerm_key_vault.secrets.id
  
  tags = {
    secret-type = "tenant-id"
    environment = var.environment
    managed-by  = "terraform"
  }
}
```

Add outputs to App Registration module (/codebase/modules/azure/app-registration/outputs.tf):

```hcl
output "client_id" {
  value     = azurerm_application.terraform.application_id
  sensitive = true
}

output "client_secret" {
  value     = azurerm_application_password.terraform.value
  sensitive = true
}

output "application_object_id" {
  value = azurerm_application.terraform.object_id
}
```

---

How the 30-Day Rotation Works
The Rotation Magic
Monthly Trigger Change:

monthly_rotation_trigger changes from "2024-01" to "2024-02" each month

This forces Terraform to create a new App Registration secret

Terraform State Management:

Terraform detects the rotate_when_changed trigger has updated

Creates a new App Registration password

Automatically updates Key Vault with the new secret

Pipeline Integration:

Next pipeline run uses the updated secret from Key Vault

Zero manual intervention required

Security Benefits
180-day maximum lifespan: Meets Azure's security requirements

30-day active rotation: Better than typical 90-day policies

Automatic compliance: Key Vault tracks expiration dates

No secret sprawl: Old secrets are automatically managed

Production Example

```hcl
# In production, you might want more control
variable "force_rotation" {
  type    = string
  default = "2024-Q1" # Change quarterly for less frequent rotation
}

# Or use a random value that changes monthly
resource "random_id" "rotation" {
  keepers = {
    rotation = formatdate("YYYY-MM", timestamp())
  }
  
  byte_length = 8
}
```

Step 3 — Comparison: WIF vs App Registration
Technical Comparison
Aspect	Workload Identity Federation	App Registration + Rotation
Setup Complexity	Moderate (Day 1 struggles)	Simple
Long-term Maintenance	Zero effort	Automated via Terraform
Security	No long-lived secrets	30-day rotation
Pipeline Impact	Failed on wrong task version	Seamless rotation
Compliance	Modern, recommended	Traditional with automation
Organizational Fit
Choose WIF when:

Starting new projects

Security team prefers token-based auth

You can invest in initial setup

Using modern Azure DevOps

Choose App Registration when:

Integrating with legacy systems

Compliance requires secret rotation tracking

Third-party tools don't support OIDC

You need immediate simplicity

Our Recommendation
For this series: We'll continue with WIF as it's the modern approach and we've overcome the initial hurdles.

For your organization: Evaluate both options. Many enterprises run a hybrid approach - WIF for new projects, App Registration for legacy systems.

---

Applying This Pattern Elsewhere
The Terraform-managed rotation pattern we built isn't just for Terraform service accounts. You can extend it to:

4.1 Service Accounts with Password Rotation
```hcl
# Rotate service account passwords monthly
resource "azuread_user" "service_account" {
  user_principal_name = "svc-terraform@yourdomain.com"
  display_name        = "Terraform Service Account"
  password            = random_password.service_account.result
}

resource "random_password" "service_account" {
  length  = 32
  special = true
  
  keepers = {
    rotation = formatdate("YYYY-MM", timestamp())
  }
}
```

4.2 Multiple App Registrations using a module
```hcl
# Generic pattern for any App Registration
module "api_app_reg" {
  source = "../app-registration"
  
  app_name       = "app-api-${var.environment}"
  environment    = var.environment
  rotation_trigger = local.monthly_rotation_trigger
}
```

What's Next (Day 4)
Repo Structure Deep Dive: Organising your codebase for scale

CAF Naming Conventions: Implementing consistent naming across environments

Environment Strategy: Dev, Test, Prod patterns that work

Tagging Standards: Cost management and operational visibility