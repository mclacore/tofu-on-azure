The guide below outlines recommended best practices to follow when using OpenTofu in Azure.

## Code

- **Use Modules**: Organize your OpenTofu code using [modules](./modules.md) promote reusability and maintainability. Modules help to encapsulate logic, reduce duplication, and simplify code.
- **Use Variables and Parameters**: Implement variables and parameters to make your code more flexible and reusable. This customization reduces duplication and eases code maintenance.
- **Use Consistent Naming Conventions**: Adopt a consistent naming convention for resources, variables, and outputs to enhance readability and manageability of your code. This helps avoid confusion and ensures a uniform structure.

<details>
<summary>Example naming conventions</summary>

| Resource Type   | Naming Convention Example |
| --------------- | ------------------------- |
| Resource Group  | `rg-<project>-<env>`      |
| Virtual Network | `vnet-<project>-<env>`    |
| Storage Account | `store<project><env>`     |

</details>

- **Implement Input Validation**: Validate input variables to ensure they meet expected criteria and prevent misconfigurations. Use validation blocks to enforce constraints on variables. Richer validation can be achieved using [JSON Schema](https://developer.hashicorp.com/terraform/language/values/variables#custom-validation-rules).

<details>
<summary>Example <code>variables.tf</code></summary>

```terraform
variable "environment" {
    description = "The environment to deploy to"
    type        = string

    validation {
        condition = contains(["dev", "staging", "production"], var.environment)
        error_message = "Invalid environment. Must be one of: dev, staging, production"
    }
}

variable "resource_name" {
    description = "The name of the resource"
    type        = string

    validation {
        condition = length(var.resource_name) <= 24
        error_message = "Resource name must be 24 characters or less"
    }
}
```

</details>

<details>
<summary>Example <code>variables.json</code></summary>

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "environment": {
      "description": "The environment to deploy to",
      "type": "string",
      "enum": ["dev", "staging", "production"],
      "errorMessage": {
        "enum": "The environment must be one of 'dev', 'staging', or 'production'."
      }
    },
    "resource_name": {
      "description": "The name of the resource",
      "type": "string",
      "maxLength": 24,
      "errorMessage": {
        "maxLength": "The resource name must be 24 characters or less."
      }
    }
  },
  "required": ["environment", "resource_name"],
  "additionalProperties": false
}
```

</details>

- **Use Locals**: Utilize locals to encapsulate complex expressions around variable manipulation, string formatting, and conditional logic. Makes your code more readable, maintainable, and reusable.

<details>
<summary>Example locals block</summary>

```terraform
locals {
    max_length        = 24
    alphanumeric_name = substr(replace(var.name, "/[^a-z0-9]/", ""), 0, local.max_length)
}

resource "azurerm_storage_account" "storage" {
    name     = local.alphanumeric_name
    location = var.location
}
```

</details>

- **Manage Dependencies**: Ensure proper sequencing and avoid circular dependencies by using `depends_on` to manage resource dependencies. Terraform providers typically handle dependencies, but `depends_on` can enforce the correct order of operations when needed.

<details>
<summary>Example <code>depends_on</code></summary>

```terraform
resource "azurerm_virtual_network" "vnet" {
    name = "my-vnet"
}

module "subnet" {
    source               = "./modules/subnet"
    name                 = "my-subnet"
    virtual_network_name = azurerm_virtual_network.vnet.name
    depends_on           = [azurerm_virtual_network.vnet]
}
```

</details>

- **Use Azure's Terraform Providers**: Leverage Azure's Terraform providers ([AzureRM](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs), [AzureAD](https://registry.terraform.io/providers/hashicorp/azuread/latest/docs), [AzureDevops](https://registry.terraform.io/providers/microsoft/azuredevops/latest/docs), [AzAPI](https://registry.terraform.io/providers/Azure/azapi/latest/docs), [AzureStack](https://registry.terraform.io/providers/hashicorp/azurestack/latest/docs)) to interact with Azure's services and APIs, enabling seamless integration and management.
- Use Workspaces: Manage multiple environments (e.g., dev, staging, production) within a single OpenTofu configuration using [workspaces](https://opentofu.org/docs/cli/workspaces/). This segregation helps manage configurations and maintain consistency across environments.

## VCS & CI/CD

- **Use Version Control**: Track changes to code and collaborate with others using a version control system like Git. This is essential for managing changes, tracking the history of OpenTofu configurations, and effective collaboration.
- **Branching Strategy**: Implement a branching strategy (e.g. GitFlow) to manage feature development, bug fixes, and releases. This ensures that code changes are organized and integrated smoothly.
- **Plan and Review Changes**: Always run `tofu plan` to review the changes before applying them. This helps to identify potential issues and understand the impact of your changes.
- **Integrate with CI/CD Pipelines**: Automate testing, building, and deployment of infrastructure by integrating the [OpenTofu GitHub Action](https://github.com/marketplace/actions/opentofu-setup-tofu) with your CI/CD pipelines. This ensures consistency, reliability, and repeatability in your deployments.

<details>
<summary>Example CI/CD pipeline</summary>

```yaml
name: OpenTofu CI/CD Pipeline

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  opentofu:
    name: "OpenTofu"
    runs-on: ubuntu-latest

    steps:
      - name: "Checkout Code"
        uses: actions/checkout@v2

      - name: "Set up Terraform"
        uses: opentofu/setup-opentofu@v1
        with:
          tofu_version: 1.8.0

      - name: "OpenTofu Format"
        run: tofu fmt -check

      - name: "OpenTofu Init"
        run: tofu init

      - name: "OpenTofu Plan"
        run: tofu plan -no-color

      - name: "OpenTofu Apply"
        run: tofu apply tfplan
```

</details>

## State management

- **Use Cloud Provisioning and Orchestration Platforms**: Leverage platforms that offer features like remote state management, collaborative tools, deployment histories, user-friendly GUI, security & scalability. This simplifies managing infrastructure as code, automating deployments, and collaborating effectively.
- **Implement State Locking**: Prevent concurrent operations that could lead to conflicts and inconsistencies by implementing state locking mechanisms.
- **Establish Backup and Restore Procedures**: Ensure data integrity and recover from failures by establishing backup and restore procedures for your state files.
- **Use Data Lookups Instead of Remote State References**: Avoid using potentially stale data and adding dependencies on your state files by opting for data lookups instead of remote state references.
- **State File Security**: Encrypt your state files to protect sensitive information. Ensure that state files are stored securely and access is restricted to authorized users.
- **Periodic State File Cleanup**: Regularly clean up- old state file versions to reduce clutter and improve performance. This helps in maintaining a clean state management system

## Documentation

- **Tag Resources**: Enhance visibility, organization, and management by tagging your resources. This helps in identifying resources, tracking costs, and managing resources effectively.

<details>
<summary>Example tags</summary>

```terraform
resource "azurerm_resource_group" "rg" {
    name     = "my-rg"
    location = "East US"
    tags = {
        environment = "dev"
        owner       = "John Doe"
    }
}
```

</details>

- **Maintain Clear Documentation**: Ensure your OpenTofu configurations are well-documented and up-to-date. This helps onboard new team members, troubleshoot issues, and manage infrastructure effectively.
- **Include Architecture Diagrams**: Provide architecture diagrams that illustrate the infrastructure setup. This helps new users to visualize the deployment and understand the relationship between resources.
- **Review OpenTofu and Azure Provider Changelogs**: Stay informed about new features, bug fixes, and improvements by regularly reviewing the [OpenTofu changelog](https://github.com/opentofu/opentofu/releases) and changelog for each Azure Terraform provider used.

## Testing

- **Include Automated Tests**: Validate your IaC to ensure correctness and prevent errors by including automated tests. This helps identify issues early, reduce risks, and improves reliability and consistency.
- **Define Rollback Strategies**: Plan for failures by defining rollback strategies to revert changes in case of errors or issues during deployments.
- **Test Infrastructure Changes in Isolation**: Test changes in a separate environment before applying them in production. This minimizes the risk of disruptions and allows for thorough testing.

## Security

- **Adopt Security Scanning Tools**: Integrate tools like [Checkov](https://www.checkov.io/7.Scan%20Examples/Terraform%20Plan%20Scanning.html) or [Terrascan](https://runterrascan.io/docs/usage/command_line_mode/) into your CI/CD pipelines to scan for security vulnerabilities and compliance violations.
- **Regular Security Audits**: Conduct regular security audits to identify and address vulnerabilities in your OpenTofu configurations. This ensures that your infrastructure remains secure over time.
- **Secure Sensitive Information**: Manage and store sensitive information such as API keys, passwords, and certificates using dedicated secrets management services like Azure Key Vault.
- **Least Privilege Principle**: Apply the principal of least privilege by granting minimal permissions required for resources. This reduces the risk of unauthorized access and enhances security.
- **Isolate Environments**: Improve security by isolating environments and using separate subscriptions, resource groups, and networks to prevent unauthorized access and reduce risks.
- **Use Sensitive Keyword**: Prevent sensitive information from being displayed in logs, state files, and other outputs by using the sensitive keyword on sensitive variables and outputs.

## Monitoring

- **Enable Logging**: Enable logging for your infrastructure resources to track changes, monitor performance, and troubleshoot issues effectively. Logs provide valuable insights into the health and behavior of your infrastructure.
- **Monitor Infrastructure**: Track performance, detect issues, and ensure availability by monitoring your infrastructure. This is crucial for identifying problems, troubleshooting issues, and optimizing performance.

<details>
<summary>Monitoring tools</summary>

| Monitoring Tool      | Description                        | Configuration Documentation                                                                            |
| -------------------- | ---------------------------------- | ------------------------------------------------------------------------------------------------------ |
| Azure Monitor        | Comprehensive monitoring solution  | [Azure Monitor](https://docs.microsoft.com/en-us/azure/azure-monitor/overview)                         |
| Log Analytics        | Log data collection and analysis   | [Log Analytics](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-overview)     |
| Application Insights | Application performance monitoring | [Application Insights](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview) |

</details>

- **Conduct Regular Reviews**: Ensure compliance, security, quality, and reliability by regularly reviewing code, monitoring infrastructure changes, and tracking deployments.
