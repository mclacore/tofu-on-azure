This is a guide for setting up the Azure CLI and OpenTofu to deploy a resource group in Azure.

## Prerequisites

- [An Azure account](https://azure.microsoft.com/en-us/free/)
- [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)

## Create Azure Service Principal

If you want to follow the [portal](https://learn.microsoft.com/en-us/entra/identity-platform/howto-create-service-principal-portal) steps, feel free. I will be covering the [CLI](https://learn.microsoft.com/en-us/cli/azure/azure-cli-sp-tutorial-1?tabs=bash) steps.

1. Log into Azure CLI and get your subscription ID (save `subscription_id` for step 3)

```bash
az login
az account list --output table
```

2. Create a service principal (save `appId`, `password`, and `tenant` for step 3)

```bash
az ad sp create-for-rbac --name "tofu-on-azure" --role owner --scopes /subscriptions/<azure_subscription_id>
```

3. Set your environment variables

```bash
export ARM_SUBSCRIPTION_ID="<azure_subscription_id>"
export ARM_TENANT_ID="<azure_subscription_tenant_id>"
export ARM_CLIENT_ID="<service_principal_appid>"
export ARM_CLIENT_SECRET="<service_principal_password>"
```

## Install OpenTofu

### Brew

```bash
brew update
brew install opentofu
```

### Standalone

```bash
# Download the installer script:
curl --proto '=https' --tlsv1.2 -fsSL https://get.opentofu.org/install-opentofu.sh -o install-opentofu.sh
# Alternatively: wget --secure-protocol=TLSv1_2 --https-only https://get.opentofu.org/install-opentofu.sh -O install-opentofu.sh

# Grant execution permissions:
chmod +x install-opentofu.sh

# Please inspect the downloaded script at this point.

# Run the installer:
./install-opentofu.sh --install-method standalone

# Remove the installer:
rm -f install-opentofu.sh
```

### PowerShell

```powershell
# Download the installer script:
Invoke-WebRequest -outfile "install-opentofu.ps1" -uri "https://get.opentofu.org/install-opentofu.ps1"

# Please inspect the downloaded script at this point.

# Run the installer:
& .\install-opentofu.ps1 -installMethod standalone

# Remove the installer:
Remove-Item install-opentofu.ps1
```

Verify version:

```bash
tofu -version
```

> [!TIP]
> If you already have Terraform and want to migrate to OpenTofu, you can find out more about that [here](https://opentofu.org/docs/intro/migration/).

## Deploy a resource group

1. Create a new directory to deploy a new resource group and `cd` into it
2. Create a file named `providers.tf` and paste the following:

```terraform
terraform {
  required_providers {
    azurerm = {
        source  = "hashicorp/azurerm"
        version = "~>3.0"
    }
    random = {
        source  = "hashicorp/random"
        version = "~>3.0"
    }
  }
}

provider "azurerm" {
  features {}
}
```

3. Create a new file named `main.tf` and paste the following:

```terraform
resource "random_pet" "rg_name" {
    prefix = var.resource_group_name_prefix
}

resource "azurerm_resource_group" "rg" {
    name     = random_pet.rg_name.id
    location = var.resouce_group_location
}
```

4. Create a new file named `variables.tf` and paste the following:

```terraform
variable "resource_group_location" {
  type        = string
  default     = "westus"
  description = "Location of the resource group."
}

variable "resource_group_name_prefix" {
  type        = string
  default     = "tofu-on-azure"
  description = "Prefix of the resource group name that's combined with a random ID so name is unique in your Azure subscription."
}
```

5. Create a new file named `outputs.tf` and paste the following:

```terraform
output "resource_group_name" {
  value = azurerm_resource_group.rg.name
}
```

6. Run `tofu init -upgrade`
   - `tofu init` is used to initialize the OpenTofu environment and download the AzureRM provider
   - `-upgrade` parameter upgrades the provider plugins to the newest version
7. Run `tofu plan -out main.tfplan`
   - `tofu plan` creates an execution plan, but doesn't execute it. It determines which actions are necessary to create the configuration specified. Important for comparing changes.
   - `-out` is an optional parameter that allows you to specify an output file for the plan, otherwise it prints to console
8. Run `tofu apply main.tfplan`
   - `tofu apply` is used to deploy the configuration
   - `tofu apply` can be ran without specifying a plan file, and it'll run on your detected `.tf` files

## Verify results

[Portal](https://portal.azure.com/#browse/resourcegroups) method.

### CLI

1. Get the Azure resource group name

```bash
resource_group_name=$(tofu output -raw resource_group_name)
```

2. Run `az group show` to display the resource group

```bash
az group show --name $resource_group_name
```

## Cleanup resources

1. Run `tofu plan` and specify `-destroy` tag

```bash
tofu plan -destroy -out main.destroy.tfplan
```

2. Run `tofu apply` to apply the execution plan

```bash
tofu apply main.destroy.tfplan
```

_Alternatively, you can run `tofu destroy` without `tofu plan/tofu apply`_
