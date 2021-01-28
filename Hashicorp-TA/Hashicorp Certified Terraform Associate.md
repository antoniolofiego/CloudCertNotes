# Hashicorp Certified Terraform Associate

# Infrastructure as code

The practice of provisioning infrastructure through software to achieve consistent and predictable environments

- Through software ⇒ Defined in code and automated
- Consistent ⇒ Repeatable and always identical
- Predictable ⇒ Exactly as specified
- Idempotency ⇒ Sets the target environment into the same configuration, regardless of the environment’s starting state
- Push/Pull Configuration ⇒ If push, the provider will push the configuration to the destination environment. If pull, the destination environment pulls the configuration from a centralized configuration server. Terraform is push, it will tell an environment how to change. Puppet and Chef  are pull, basically having the managed system poll a central server to see if there are updates.

Configurations can be versioned and stored in source control repositories.

- Imperative

    Describe the specific steps in which something is implemented 

- Declarative

    Describe the logic of the computation without specifying the implementation details

# Hands on with Terraform

## Terraform Components

- Terraform Executable

    The CLI that you interact with

- Terraform files with extension `.tf`

    The actual file that hold our configurations and code

- Terraform plugins

    Pieces of software that allow Terraform to interact with infrastructure providers such as AWS, Azure and GCP

- Terraform state

    A file that keeps track of what has been created and deployed. Maintains a log of the current state of the infrastructure

## Terraform terminology

- Variables

    Value holders that can be used to customize a configuration before execution. They can be defined everywhere in the code but it is suggested to keep them in a separate file called `terraform.tfvars` that can be referenced at later stage.

    A variable block is defined like:

    ```yaml
     variable "aws_region" {
      description = "AWS region"
      type        = string
      default     = "us-west-2"
    }
    ```

    There are three optional argument in the variable declaration:

    - Description, to describe the purpose
    - Type, data type held in the variable
    - Default, default value assigned if no other value is declared
- Providers

    Terraform relies on plugins called "providers" to configure resources offered by remote systems. These can be cloud providers (such as AWS, Azure, GCP), tools, or platforms (such as Kubernetes, Grafana, PagerDuty). Each provider has extensive documentation with requirements and resources

- Data Sources

    Information provided by providers, such as AMIs and AZs in AWS, that Terraform can request as needed 

- Resources

    Infrastructure objects managed by Terraform and deployed on the cloud platform.

- Output

    Information and data obtained once the resources are deployed, such as public IPs of instances, DNS records and such