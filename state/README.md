OpenTofu state is used to reconcile deployed resources with the desired state. State allowes OpenTofu to know what Azure resources to add, update, or delete.

By default, OpenTofu state is stored locally, which isn't ideal for a few reasons:

- Local state doesn't work well in a team or collaborative environment
- OpenTofu state can include sensitive information
- Storing state locally increases chance of accidental deletion

## Prerequisites

- Completed [setup](../setup/README.md)
- Set your environment variables

```bash
export ARM_SUBSCRIPTION_ID="<azure_subscription_id>"
export ARM_TENANT_ID="<azure_subscription_tenant_id>"
export ARM_CLIENT_ID="<service_principal_appid>"
export ARM_CLIENT_SECRET="<service_principal_password>"
```

## Configure remote state storage account

Assuming you have OpenTofu configured at this point, let's configure our remote state storage accoung using it!

1. Create a new directory for your remote state storage account
2. Create a new file named `main.tf` in the directory and paste the following:

```terraform
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~>3.0"
    }
  }
}

provider "azurerm" {
  features {}
}

resource "random_string" "resource_code" {
  length  = 5
  special = false
  upper   = false
}

resource "azurerm_resource_group" "tfstate" {
  name     = "tfstate"
  location = "westus"
}

resource "azurerm_storage_account" "tfstate" {
  name                     = "tfstate${random_string.resource_code.result}"
  resource_group_name      = azurerm_resource_group.tfstate.name
  location                 = azurerm_resource_group.tfstate.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
  allow_nested_items_to_be_public = false
}

resource "azurerm_storage_container" "tfstate" {
  name                  = "tfstate"
  storage_account_name  = azurerm_storage_account.tfstate.name
  container_access_type = "private"
}
```

3. Run `tofu init` then `tofu apply` to configure the Azure storage account and container

## Configure remote state backend

To configure the backend state, you need the following Azure storage information:

- `storage_account_name`: The name of the storage account
- `container_name`: The name of the container
- `access_key`: The access key of the storage account
- `key`: The name of the state store file to be created
