## Modules

- **Avoid Monolithic Modules**: Design modules with a narrow scope focused on a single cloud resource and its dependencies. This increases reusability and limits the potential impact of changes.
- **Encode Expertise in Modules**: Build modules that incorporate your company's requirements, security policies, and standard. Avoid generic modules that pass through values without adding configuration expertise.
- **Pin Module and Provider Versions**: Ensure infrastructure reproducibility and stability by pinning module and provider versions to specific versions or version ranges.

## Examples

### Narrow scoping

<details open>
<summary>Do:</summary>

```terraform
module "vnet" {
  source = "./modules/vnet"
  name   = "my-vnet"
}

module "storage" {
  source = "./modules/storage"
  name   = "my-storage"
}

module "vm" {
  source = "./modules/vm"
  name   = "my-vm"
}
```

</details>
<details open>
<summary>Don't do:</summary>

```terraform
module "my-module" {
  source       = "./modules/my-module"
  vnet_name    = "my-vnet"
  storage_name = "my-storage"
  vm_name      = "my-vm"
}
```

</details>

### Opinionated modules

<details open>
<summary>Do:</summary>

```terraform
resource "azurerm_storage_account" "storage" {
    name                          = var.name
    account_kind                  = "StorageV2"
    account_tier                  = "Standard"
    access_tier                   = "Hot"
    min_tls_version               = "TLS1_2"
    replication_type              = var.replication_type
    public_network_access_enabled = var.public_network_access_enabled
}
```

```terraform
module "storage" {
    source                        = "./modules/storage"
    name                          = "my-storage"
    replication_type              = "LRS"
    public_network_access_enabled = false
}
```

</details>
<details open>
<summary>Don't do:</summary>

```terraform
resource "azurerm_storage_account" "storage" {
    name                          = var.name
    account_kind                  = var.account_kind
    account_tier                  = var.account_tier
    access_tier                   = var.access_tier
    min_tls_version               = var.min_tls_version
    replication_type              = var.replication_type
    public_network_access_enabled = var.public_network_access_enabled
}

```

```terraform
module "storage" {
    source                        = "./modules/storage"
    name                          = "my-storage"
    account_kind                  = "StorageV2"
    account_tier                  = "Standard"
    access_tier                   = "Hot"
    min_tls_version               = "TLS1_2"
    replication_type              = "LRS"
    public_network_access_enabled = false
}
```

</details>

### Pin versions

<details open>
<summary>Do:</summary>

```terraform
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~>2.0.0"
    }
  }
}
```

```terraform
module "storage" {
    source                        = "github.com/mclacore/tofu-on-azure//modules/storage?ref=abcdefg"
    name                          = "my-storage"
    replication_type              = "LRS"
    public_network_access_enabled = false
```

</details>
<details open>
<summary>Don't do:</summary>

```terraform
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
    }
  }
}
```

```terraform
module "storage" {
    source                        = "github.com/mclacore/tofu-on-azure//modules/storage"
    name                          = "my-storage"
    replication_type              = "LRS"
    public_network_access_enabled = false
}
```

</details>
