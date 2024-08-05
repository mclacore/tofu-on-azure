## Modules

- Avoid monolithic modules (e.g. a single module that creates a VNet, Storage Account, and VM). Modules should be narrowly scoped to a single cloud resources and its dependencies. This makes it more re-usable, and limits the potential blast radius of changes.
- Modules should encode expertise, not simply pass-through values. Many of the official Terraform modules are simply pass-throughs, and don't provide much value as there is no _configuration_ expertise baked in. Instead, they are quite **generic**. Generic modules burden the user with the responsibility of knowing how to configure the underlying resource. Encode your company's requirements, security policies, and standards into your module. Make them **opinionated**.
- Pin your module and provider versions. This is a best practice to ensure that your infrastructure is reproducible and doesn't break due to changes in the module or provider.

## Examples

### Narrow scoping

<details open>
<summary>Do</summary>

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
<summary>Don't do</summary>

```terraform
module "my-module" {
  source = "./modules/my-module"
  vnet_name    = "my-vnet"
  storage_name = "my-storage"
  vm_name      = "my-vm"
}
```

</details>

### Opinionated modules

<details open>
<summary>Do</summary>

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
<summary>Don't do</summary>

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
<summary>Do</summary>

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
<summary>Don't do</summary>

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
