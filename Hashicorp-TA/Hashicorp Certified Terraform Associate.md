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

# Terraform Components

- Terraform Executable

    The CLI that you interact with

- Terraform files with extension `.tf`

    The actual file that hold our configurations and code

- Terraform plugins

    Pieces of software that allow Terraform to interact with infrastructure providers such as AWS, Azure and GCP

- Terraform state

    A file that keeps track of what has been created and deployed. Maintains a log of the current state of the infrastructure

# Terraform Terminology

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

- State

    A JSON formatted file that Terraform builds and maintains automatically to keep track of the current configuration. It includes metadata on specifics about the deployed resources, state versions and other key measures. 

    Certain backends support state locking, meaning that no write operations are allowed to be performed on that specific configuration until the locks are lifted, usually between deployments. 

    State is always stored at least locally in the same folder where your Terraform configuration files are stored but it can be also stored remotely (AWS, Azure, GCP, Terraform Cloud).

    Terraform has the concept of workspaces where each working environment has its own separate state file and when switching between different workspaces Terraform is aware of which workspace it is operating in and references the correct state.

    The state file includes

    - `version`: the version of the state file
    - `terraform_version`: the version of Terraform that created that file
    - `serial`: a number that increases every time the state file is updated
    - `lineage`: a unique ID assigned to a state when it is created
    - `outputs`: outputs at the end of a deployment
    - `resources`: list of all the resources that the Terraform configuration has deployed
- Plan

    A process where Terraform inspects the current state of the configuration, creates a dependency graph to understand the order in which resources can be deployed, calculates additions of new resources and deletions or modifications of existing resources, and finally executes the steps to reach the defined end state. Plans can be saved as `.tfplan` files and applied at later stages or can be applied right away. 

# Terraform Syntax

Terraform uses a specific language for its configuration files, the **HashiCorp Configuration Language** (HCL), which is used in multiple HashiCorp products. It has several benefits over other commonly used file types such as JSON because it is more readable and easily editable, it supports functions, conditionals and other expressions to support more intricate and customizable configurations.

## Object types

- number = `2`
- string = `"hello"`
- bool = `true`
- list = `[ "ubuntu", "amazon-linux" ]`
- map = `{ id = 4, name = "t2.micro", active = true }`

## References

Reserved keywords in Terraform. The general syntax is `keyword.reference_name.attribute`

- `var` used to reference variables
- `resource_type` (such as `aws_instance`) to refer to specific resources
- `local` for local values in the configuration file
- `module` to reference Terraform modules

## Variable interpolation

Used to insert the value of a reference in a string (think about string literals in JS)

`name = "my-${var.instance-name}"`  

## Retrieving values from lists, maps and data sources

`local.list_of_values[4]` returns the 5th element of the list

`local.map_of_values["name"]` returns the value at the key `"name"` in the map

`data.data_source.full.path.to.resource` to return the specific value from a data source (for example `data.aws_availability_zones.azs.names[0]`)

## Blocks

```yaml
block_type "label_one" "label_two" {

	key = value
	key = value
	
	block {
		key = value
	}

}
```