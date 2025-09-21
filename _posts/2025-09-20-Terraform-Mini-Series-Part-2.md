---
layout: post
comments: true
thumb: Terraform_on_Azure.jpeg
smallthumb: Terraform
title: "Terraform on Azure DevOps, Day 2: Locking State & Managing Secrets with Azure Storage and Key Vault"
tagline: Learn Terraform in a mini series for Azure and M365
description: "Secure your Terraform pipeline. Learn to enforce Azure Blob state locking, wire Azure DevOps variable groups to Key Vault for dynamic secrets, and implement least-privilege RBAC for a robust multi-environment foundation."
slug: terraform-mini-series-part-2
tags: [terraform, azure, azure-devops, pipelines, oidc, key-vault, rbac, state, storage, m365]
canonical_url: https://anothertechieblog.co.uk/terraform-mini-series-part-2
modified: 2025-09-20
---

> **Series goal (reminder):** Stand up a practical, multi‑environment Terraform platform on Azure DevOps (with split pipelines for Infra/Entra/MS Graph), using secure auth, remote state, and reusable modules—scaling from *Dev* to *Prod*.

# Day 2 — Locking State & Managing Secrets with Azure Storage and Key Vault

---

## What you’ll build today

- **Secure State Management**: Bring the Azure Storage account and container you created in Day 1 **under Terraform management** (via import) and protect them with a **CanNotDelete** lock.
- **Centralised Secrets**: Create an **Azure Key Vault** as the single source of truth for pipeline secrets and sensitive variables.
- **Dynamic Secrets in DevOps**: Link the Key Vault to an **Azure DevOps Variable Group** so pipelines fetch the **latest secret values at runtime**—no secrets in YAML.
- **Least‑Privilege Access**: Assign precise RBAC on Storage & Key Vault to the pipeline identity used by your **Workload Identity Federation (OIDC)** service connection.

> **Day 1 recap**: You already have a repo + pipeline authenticating to Azure via **OIDC** (no client secrets) and a remote state backend in **Azure Storage** with **AAD/OIDC** and **blob lease locking**. ([Day 1 post])

---

## Why manage these resources via code?

Infrastructure that isn’t codified drifts—someone clicks a setting, policies change, and suddenly your pipeline fails. **Infrastructure as Code (IaC)** fixes that: every change is reviewed, versioned, and reproducible. Moving your Day 1 bootstrap (state storage) **into Terraform** means its security posture (versioning, retention, and locks) is **enforced**. If disaster strikes, you can rebuild environments from Git.

> The Terraform **azurerm** backend stores state in Azure Blob Storage and uses Blob’s native **lease‑based locking**—safe for teams/CI. With `use_azuread_auth = true` it authenticates to the blob **data plane** using Entra ID (Azure AD).  
> • Backend docs: <https://developer.hashicorp.com/terraform/language/backend/azurerm>  
> • Ensure your pipeline identity has **Storage Blob Data Contributor** on the state account/container (data plane), otherwise `terraform init` will fail.

---

## Step 0 — Prereqs

- Azure subscription + permissions to assign roles and create resources.  
- Azure DevOps org/project + **ARM service connection** using **Workload Identity Federation** (from Day 1).  
- The state RG/storage/container from Day 1 (names you actually used).

---

## Step 1 — Import the state storage into Terraform (and lock it)

We’ll create a small **core‑infra** module, then **import** the existing RG/Storage/Container so Terraform manages them going forward.

> **What terraform import does**: it associates an existing cloud resource with a resource block in your configuration, adding it to Terraform **state** without changing it. From then on, Terraform tracks it for drift and enforces configuration. You still need to make your config **match the real resource** (same names/IDs), or Terraform will want to replace it during an apply.

### 1.1 Module layout

```
/codebase
  /modules
    /azure
      /core-infra
        main.tf
        variables.tf
        outputs.tf
```

**`/codebase/modules/azure/core-infra/variables.tf`**
```hcl
variable "environment"      { type = string }
variable "location" { 
    type = string
    default = "UK South" 
}
variable "subscription_id"  { type = string }
variable "state_container"  { 
    type = string
    default = "tfstate" 
}
variable "state_rg_name"    { type = string }
variable "state_sa_name"    { type = string }
```

**`/codebase/modules/azure/core-infra/main.tf`**
```hcl
resource "azurerm_resource_group" "state" {
  name     = var.state_rg_name
  location = var.location
  tags     = { environment = var.environment, managed-by = "terraform" }
}

resource "azurerm_storage_account" "state" {
  name                     = var.state_sa_name               # must match existing SA name
  resource_group_name      = azurerm_resource_group.state.name
  location                 = azurerm_resource_group.state.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
  allow_nested_items_to_be_public = false

  blob_properties {
    versioning_enabled  = true
    change_feed_enabled = true
    delete_retention_policy { days = 30 }
  }

  tags = { environment = var.environment, managed-by = "terraform" }
}

resource "azurerm_storage_container" "state" {
  name                  = var.state_container                # e.g., tfstate
  storage_account_id    = azurerm_storage_account.state.id   # ARM path style
  container_access_type = "private"
}

resource "azurerm_management_lock" "state_storage_lock" {
  name       = "cannot-delete"
  scope      = azurerm_storage_account.state.id
  lock_level = "CanNotDelete"
  notes      = "Protect Terraform state storage."
}
```

**`/codebase/modules/azure/core-infra/outputs.tf`**
```hcl
output "state_storage_rg_name"         { value = azurerm_resource_group.state.name }
output "state_storage_account_name"    { value = azurerm_storage_account.state.name }
output "state_storage_container_name"  { value = azurerm_storage_container.state.name }
```

### 1.2 Bootstrap config and imports

```
/codebase/env/bootstrap/
```

**`/codebase/env/bootstrap/main.tf`**
```hcl
terraform {
  required_providers {
    azurerm = { source = "hashicorp/azurerm", version = "~> 4.0" }
  }
  backend "azurerm" {
    resource_group_name  = "rg-tfstate-core-uks"   # your actual names
    storage_account_name = "sttfstate9683"
    container_name       = "tfstate"
    key                  = "bootstrap/core.tfstate"
    use_azuread_auth     = true
  }
}

provider "azurerm" { 
    features {} 
    subscription_id = var.subscription_id
    }

module "core_infra" {
  source           = "../../modules/azure/core-infra"
  environment      = "bootstrap"
  subscription_id  = var.subscription_id
  state_rg_name    = "rg-tfstate-core-uks"
  state_sa_name    = "sttfstate9683"
  state_container  = "tfstate"
}
```

**`/codebase/env/bootstrap/variables.tf`**
```hcl
variable "subscription_id" { type = string }
```

**Run the imports** (locally):

You'll need Azure CLI installed on your machine https://learn.microsoft.com/en-us/cli/azure/authenticate-azure-cli-interactively?view=azure-cli-latest to do it from within Visual Studio Code which is the approach I've taken. 

> winget install Microsoft.AzureCLI 

after installing, you will need to close all of the visual studio windows open for VSCode to pick up the new path variable for az login or you end up with an error saying I don't know what you mean. 


```bash
az login
cd codebase/env/bootstrap/
terraform init

terraform import -var "subscription_id=50b57618-d85f-4203-826a-2e464126e24b" module.core_infra.azurerm_resource_group.state /subscriptions/50b57618-d85f-4203-826a-2e464126e24b/resourceGroups/rg-tfstate-core-uks

terraform import `
  -var "subscription_id=50b57618-d85f-4203-826a-2e464126e24b" `
  module.core_infra.azurerm_storage_account.state `
  "/subscriptions/50b57618-d85f-4203-826a-2e464126e24b/resourceGroups/rg-tfstate-core-uks/providers/Microsoft.Storage/storageAccounts/sttfstate9683"


# Storage container import: provider versions differ
# Try ARM path first; if it fails, use the blob URL form
terraform import `
  -var "subscription_id=50b57618-d85f-4203-826a-2e464126e24b" `
  module.core_infra.azurerm_storage_container.state `
  "/subscriptions/50b57618-d85f-4203-826a-2e464126e24b/resourceGroups/rg-tfstate-core-uks/providers/Microsoft.Storage/storageAccounts/sttfstate9683/blobServices/default/containers/tfstate" 
# or
terraform import module.core_infra.azurerm_storage_container.state   https://sttfstate9683.blob.core.windows.net/tfstate

Now we will run a plan, review whats going to change and then apply the changes if we are happy

terraform plan -var "subscription_id=50b57618-d85f-4203-826a-2e464126e24b"

After running the apply stage you will need to confirm the changes by entering yes.

terraform apply -var "subscription_id=50b57618-d85f-4203-826a-2e464126e24b"
```

I also got this error whilst running this: 

![TerraformOutputFailureBlob.png](/assets/images/2025-09-20-Terraform-Mini-Series-Part-2/TerraformOutputFailureBlob.png)

some troubleshooting through the Azure CLI returned the problem, I needed to have more permissions over the storage, one of the roles defined in the message. I opted for Storage Blob Container Contributor. 

![TerraformOutputFailureBlobRoles.png](/assets/images/2025-09-20-Terraform-Mini-Series-Part-2/TerraformOutputFailureBlobRoles.png)

> **Notes on container import:** Recent provider versions sometimes accept the **blob URL** format when the ARM‑path style fails. See provider docs/issues for details.  
> • Storage container resource: <https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/storage_container>  
> • Import syntax discussion: <https://github.com/hashicorp/terraform-provider-azurerm/issues/29065>

---

We have now successfully imported the resource group, storage account & container for the statefiles to be manged under terraform as well.

## Step 2 — Create Key Vault (RBAC), assign permissions, add a demo secret (all in Terraform)

In many organisations, **RBAC** is the standard. We’ll create a Key Vault **with RBAC**, grant the pipeline identity read access to **secrets**, and add a small **demo secret** to prove the wiring—everything managed by Terraform.

> If your org prefers the legacy **Access policy** model for ADO Variable Groups, you can flip the approach: set `enable_rbac_authorization = false` and replace the role assignment with an `azurerm_key_vault_access_policy` that grants **Get, List**.

Add to `/codebase/modules/azure/core-infra/main.tf`
```hcl
data "azurerm_client_config" "current" {}

variable "pipeline_principal_id" {
  type        = string
  description = "Object ID of the Azure DevOps service connection principal"
}

resource "azurerm_key_vault" "secrets" {
  name                          = "kv-secrets-${var.environment}-uks"
  location                      = azurerm_resource_group.state.location
  resource_group_name           = azurerm_resource_group.state.name
  tenant_id                     = data.azurerm_client_config.current.tenant_id
  sku_name                      = "standard"
  enable_rbac_authorization     = true   # RBAC model
  purge_protection_enabled      = true
  soft_delete_retention_days    = 90
  public_network_access_enabled = true
  tags = { environment = var.environment, managed-by = "terraform" }
}

# Grant the pipeline SP read access to secrets (list/get)
resource "azurerm_role_assignment" "kv_secrets_user" {
  scope                = azurerm_key_vault.secrets.id
  role_definition_name = "Key Vault Secrets User"
  principal_id         = var.pipeline_principal_id
}

# Demo secret to prove end-to-end flow (non-sensitive example)
resource "azurerm_key_vault_secret" "demo" {
  name         = "tfstate-storage-account-name"
  value        = var.state_sa_name
  key_vault_id = azurerm_key_vault.secrets.id
}
```

**Pass the pipeline principal ID in `/codebase/env/bootstrap/main.tf`**
```hcl
module "core_infra" {
  # ...existing args...
  pipeline_principal_id  = "YOUR-AZURE-DEVOPS-SERVICE-CONNECTION-OBJECT-ID"
}
```

Run terraform apply again from the bootstrap/ directory to create the Key Vault, assign RBAC, and add the demo secret.

> **RBAC role explanation:** The **Key Vault Secrets User** role allows list/get on secrets in an RBAC‑mode vault (data‑plane). See: <https://learn.microsoft.com/azure/key-vault/general/rbac-guide>

---

## Step 3 — Link Key Vault to Azure DevOps (with RBAC) and use the secret at runtime

1. **Pipelines → Library → + Variable group** (e.g., tf-bootstrap-kv).  
2. Toggle **Link secrets from an Azure key vault as variables**.  
3. Choose your **ARM service connection** (WIF) and **Authorize**.  
4. Select your **Key Vault** and **Authorize**.  
5. **Add** the secret `tfstate-storage-account-name` and **Save**.

**Use it in the pipeline:**
```yaml
# azure-pipelines.yml
variables:
- name: envName
  value: dev
- group: tf-dev           # non-secret config (from Day 1)
- group: tf-bootstrap-kv  # secrets from Key Vault (linked)

stages:
- stage: PlanApply
  jobs:
  - job: plan_apply
    steps:
    - task: TerraformTask@5
      displayName: terraform init (azurerm backend + OIDC)
      inputs:
        command: init
        workingDirectory: codebase/env/$(envName)
        backendType: azurerm
        backendAzureRmUseEntraIdForAuthentication: false
        backendAzureRmUseCliFlagsForAuthentication: true
        backendServiceArm: 'sc-ado-terraform-wif'
        backendAzureRmResourceGroupName: $(tfstateRg)
        backendAzureRmStorageAccountName: $(tfstate-storage-account-name) # now from KV
        backendAzureRmContainerName: $(tfstateContainer)
        backendAzureRmKey: $(tfstateKey)
```

> **How Variable Groups ↔ Key Vault works**: Only **secret names** are mapped; **values** are fetched at **runtime**. Updates to existing secret **values** flow automatically; adding/removing **new secret names** requires updating the Variable Group’s selected list.  
> **Important doc note:** Microsoft’s page currently says **RBAC‑mode vaults aren’t supported** for this Variable Group feature. In our testing, RBAC **did** work when the pipeline principal had **Key Vault Secrets User**; behavior may vary by tenant/rollout. If it fails for you, switch to **Access policies**, or keep **RBAC** and fetch with the AzureKeyVault@2 task instead.  
> • ADO docs (support note): <https://learn.microsoft.com/azure/devops/pipelines/library/link-variable-groups-to-key-vaults?view=azure-devops>  
> • Key Vault RBAC guide: <https://learn.microsoft.com/azure/key-vault/general/rbac-guide>

---

## Troubleshooting

- **`terraform init` fails to reach state** (403 or lease errors): ensure the pipeline principal has **Storage Blob Data Contributor** on the state account/container; confirm backend has `use_azuread_auth = true`
- **Variable Group can’t select secrets**: confirm the pipeline principal holds **data‑plane** permissions (RBAC: *Key Vault Secrets User*) and the secret isn’t expired/disabled. If still blocked, use **Access policies** or the **AzureKeyVault@2** task.
- **Import drift**: make sure your Terraform arguments (names/IDs) **exactly match** the existing resources before you import; computed names will cause replacement.
- **Container import**: if the ARM path fails, use the **blob URL** syntax shown above.

---

## What’s next (Day 3)

- **Multi‑environment structure** (dev/test/prod) with reusable modules and `*.tfvars`
- **Promotion** patterns and environment‑scoped Key Vaults/Variable Groups.
- **CAF‑aligned naming** for consistency across your estate.
