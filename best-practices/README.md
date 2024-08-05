The guide below outlines recommended best practices to follow when using OpenTofu in Azure.

## Code

- Break down code into reusable modules for easier management, organization, and scalability. Promotes reusability and maintainability, reduces duplication, and makes code easier to understand.
- Writing with IaC to define your infrastructure ensures consistency, repeatability, and predictability in your deployments.
- Use variables and parameters to make your code more flexible and reusable. Helps to customize deployments, reduce duplication, and make code easier to maintain.
- Manage dependencies between resources using `depends_on` to ensure proper sequencing and avoid circular dependencies.
- Use Azure's Terraform providers ([AzureRM](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs), [AzureAD](https://registry.terraform.io/providers/hashicorp/azuread/latest/docs), [AzureDevops](https://registry.terraform.io/providers/microsoft/azuredevops/latest/docs), [AzAPI](https://registry.terraform.io/providers/Azure/azapi/latest/docs), [AzureStack](https://registry.terraform.io/providers/hashicorp/azurestack/latest/docs)) to interact with Azure's services and APIs, enabling seamless integration and management.

## VCS & CI/CD

- Use version control system like Git to track changes to code and collaborate with others. A necessity to manage changes, track history of OpenTofu configurations, and collaborate effectively.
- Integrate OpenTofu with your CI/CD pipelines to automate testing, building, and deployment of infrastructure. Paramount to ensure consistency, reliability, and repeatability in your deployments.

## State management

- Use a cloud provisioning and orchestration platform that provides features like remote state management, collaborative tools, deployment histories, user-friendly GUI, security & scalability. Makes it easier to manage infrastructure as code, automate deployments, and collaborate effectively.
- Implement state locking mechanisms to prevent concurrent operations that would lead to conflicts and inconsistencies in your deployments.
- Establish backup and restore procedures for your OpenTofu state files to prevent data loss, recover from failures, and maintain data integrity.

## Documentation

- Tag your resources to enhance visibility, organization, and management of resources. Recommended to identify resources, track costs, and manage resources effectively.
- Maintain clear and up-to-date documentation to help others understand your OpenTofu configurations. Helps to onboard new team members, troubleshoot issues, and manage infrastructure effectively.
- Regularly review OpenTofu changelog, release notes, and updates to stay informed about new features, bug fixes, and improvements.

## Testing

- Include automated tests to validate your IaC, ensure correctness, and prevent errors. Testing is vital to identify issues early, reduce risks, and improve reliability and consistency.
- Define rollback strategies to revert changes in case of failures, errors, or issues during deployments.

## Security

- Follow security best practices when writing OpenTofu code to protect sensitive information and prevent unauthorized access.
- Conduct regular audits to identify non-compliance issues, security vulnerabilities, and areas for improvement to maintain a secure environment.
- Manage and store sensitive information (e.g. API keys, passwords, certificates) securly using a dedicated secrets management service like Azure Key Vault.
- Isolate environments using separate subscriptions, resource groups, and networks to prevent unauthorized access, reduce risks, and improve security.

## Monitoring

- Monitor your infrastructure to track performance, detect issues, and ensure availability. It's crucial to identify problems, troubleshoot issues, and optimize performance.
- Regularly conduct code reviews, monitor infrastructure changes, and track deployments to ensure compliance, security, quality, and reliability in your configurations.
