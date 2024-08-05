OpenTofu state is used to reconcile deployed resources with the desired state. State allows OpenTofu to know what Azure resources to add, update, or delete.

By default, OpenTofu state is stored locally, which isn't ideal for a few reasons:

- Local state doesn't work well in a team or collaborative environment
- OpenTofu state can include sensitive information
- Storing state locally increases chance of accidental deletion

## Prerequisites

- Completed [setup](../setup/README.md)

## Configure remote state storage account

Assuming you have OpenTofu configured at this point, let's configure our remote state storage account using it!

1. Create a new directory for your remote state storage account and `cd` into it
2. Create a new file named `providers.tf` in the directory and paste the following:

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
```

3. Create a new file named `main.tf` in the directory and paste the following:

```terraform
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

4. Run `tofu init` then `tofu apply` to configure the Azure storage account and container

## Configure remote state backend

To configure the backend state, you need the following Azure storage information:

- `storage_account_name`: The name of the storage account
- `container_name`: The name of the container
- `key`: The name of the state store file to be created

1. Fetch storage account name with the following command:

```bash
az storage account list --resource-group tfstate --query "[].name" --output tsv
```

2. Once you have the required information, create a new file named `backend.tf` in the directory and paste the following:

```terraform
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~>3.0"
    }
  }
  backend "azurerm" {
      resource_group_name  = "tfstate"
      storage_account_name = "<storage_account_name>"
      container_name       = "tfstate"
      key                  = "terraform.tfstate"
  }

}

provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "state-demo-secure" {
  name     = "state-demo"
  location = "westus"
}
```

3. Run `tofu init` then `tofu apply` to configure the backend state

4. Confirm the state is being stored by running the following:

```bash
az storage blob list --account-name <storage_account_name> --container-name tfstate
```

> [!NOTE]
> Azure blobs are automatically locked before any operation that writes state. This pattern prevents concurrent state operations, which can cause corruption. For more info, see [state locking](https://www.terraform.io/docs/state/locking.html)

> [!NOTE]
> Data stored in an Azure blob is encrypted before being persisted. When needed, the OpenTofu retrieves the state from the backend and stores it in local memory. If this pattern is used, state is never written to your local disk. For more info about Azure storage encryption, see [Azure storage encryption for data at rest](https://learn.microsoft.com/en-us/azure/storage/common/storage-service-encryption)
