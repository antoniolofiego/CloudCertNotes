# Hashicorp Certified Terraform Associate

# Infrastructure as code

The practice of provisioning infrastructure through software to achieve consistent and predictable environments

- Through software ⇒ Defined in code and automated
- Consistent ⇒ Repeatable and always identical
- Predictable ⇒ Exactly as specified
- Idempotency ⇒ Sets the target environment into the same configuration, regardless of the environment’s starting state
- Push/Pull Configuration ⇒ If push, the provider will push the configuration to the destination environment. If pull, the destination environment pulls the configuration from a centralized configuration server. Terraform is push, it will tell an environment how to change. Puppet and Chef are pull, basically having the managed system poll a central server to see if there are updates.

Configurations can be versioned and stored in source control repositories.

- Imperative

    Describe the specific steps in which something is implemented 

- Declarative

    Describe the logic of the computation without specifying the implementation details

# Terraform Components

- Terraform Executable

    The CLI that you interact with

- Terraform files with extension `.tf`

    The actual file that hold our configurations and code. When Terraform is run in an folder, it will look for every `.tf` file in that directory and stitch them together. This helps with code splitting for easier management.

- Terraform plugins

    Pieces of software that allow Terraform to interact with infrastructure providers such as AWS, Azure and GCP

- Terraform state

    A file that keeps track of what has been created and deployed. Maintains a log of the current state of the infrastructure

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

```
block_type "label_one" "label_two" {
  key = value
  key = value

  block {
    key = value
  }
}
```

## Functions

Similar function syntax to most programming languages

`function(argument1, argument2, ...)`

Arguments are not named, so the position in which they are called is important

# Terraform Terminology

## Variables

Value holders that can be used to customize a configuration before execution. They can be defined everywhere in the code but it is suggested to keep them in a separate `.tfvars` file that can be referenced at later stage. Variables can be defined in environments, in files and in the command line.

An environment variable block is declared as:

```yaml
 variable "aws_region" {
  description = "AWS region"   // variable purpose
  type        = string         // data type of the variable
  default     = "us-west-2"    // default value if no other value is declared when used
}
```

A file variable line is simply declared as:

```yaml
aws_region = "us-west-2"
```

In the command line, a variable is declared as:

```yaml
terraform plan -var 'aws_region=us-west-2' 
```

### Variable declaration and overriding

Variables can be defined in multiple parts of our configuration, so Terraform works on a variable precedence system that overrides variables that are defined multiple times in different locations. 

In order from least to highest priority, Terraform has `environment` → `file` →  `command line`

## Providers

Terraform relies on plugins called "providers" to configure resources offered by remote systems. These can be cloud providers (such as AWS, Azure, GCP), tools, or platforms (such as Kubernetes, Grafana, PagerDuty). Each provider has extensive documentation with requirements and resources

## Data Sources

Information provided by providers, such as AMIs and AZs in AWS, that Terraform can request as needed 

## Resources

Infrastructure objects managed by Terraform and deployed on the cloud platform.

### Resources Arguments

Optional arguments that are specified along the resource to allow for complex configuration.

- `depends_on`: Used to handle hidden resource or module dependencies that Terraform can't automatically infer
- `count`: Accepts a whole number, and creates that many instances of the resource or module
- `for_each`: Accepts a map or a set of strings, and creates an instance for each item in that map or set. Once a `for_each` argument is set, the resource has access to a new `each` object with two attributes, `key` and `value`. If a map is provided, `key` and `value` are each key and value of the map. If a set of strings is provided, `key` and `value` are the same.
- `Providers`: Specifies which provider configurations from the parent module will be available inside the child module.

## Output

Information and data obtained once the resources are deployed, such as public IPs of instances, DNS records and such

## State

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

## Plan

A process where Terraform inspects the current state of the configuration, creates a dependency graph to understand the order in which resources can be deployed, calculates additions of new resources and deletions or modifications of existing resources, and finally executes the steps to reach the defined end state. Plans can be saved as `.tfplan` files and applied at later stages or can be applied right away. 

## Provisioners

A Terraform functionality to perform post-deployment configuration. It is not recommended by HashiCorp and should be used exclusively as a last resort approach. Provisioners can be local or remote, can run during a resource creation or destruction and a single resource can have multiple provisioners.

## Functions

A set of built-in functions that perform various tasks. Functions can be tested in the `terraform console`. There are various categories of functions:

- Numeric
- String
- Collection
- Encoding
- Fileystem
- Date and Time
- Hash and Crypto
- IP Network
- Type Conversion

## Workspaces

A Terraform-native way of managing different environments with different states. Environments could be created by hand, specifying the folders that configurations should target. Workspaces handle the complexities for you and can be created via the CLI using:

```yaml
terraform workspace new dev
```

with `dev` being the name of the workspace. Terraform will switch to the `dev` context and the next runs of `terraform plan` or `terraform apply` will use this context. You can retrieve the environment name from a variable called `terraform.workspace` and use it as a local variable in your configurations.

To switch to a different (already existing) environment, use:

```yaml
terraform workspace select prod
```

That will switch to the `prod` environment, and `plan`ning and `apply`ing will affect only the `prod` context, leaving the state of the `dev` environment unchanged.

## Modules

Modules are reusable components that encapsulate commonly used configurations or any combination of resources and code that you would like to reuse in multiple projects. Modules support versioning and different versions of a module can be invoked as necessary. A module automatically inherits the default provider configurations from its parent (be it another module or the root module that houses your basic configuration). Modules don't currently support resource arguments like `count` or `for_each`. 

### Consuming Modules

There are a multitude of commonly used infrastructure configurations at [https://registry.terraform.io](http://registry.terraform.io). In our configuration file, we refer to specific modules using the `source` argument:

```
module "module-tag" {

  source  = "module_name" or "./module_location"
  version = "1.0.0"

  arg1 = value1
	arg2 = value2
	...
}
```

It is strongly suggested to always specify the version of a module. When it is not specified, Terraform will always retrieve the latest version of the module and it is possible that something breaks between versions. Specifying a module version will avoid unexpected or unwanted changes.

### Building Modules

Modules are made of input variables, resources and output variables. When building a module, you want to make sure to collect the information needed for that module to work in the form of input variables and output the wanted/needed resources and data. Building modules is as easy as creating a separate configuration file with its own outputs, variables and resources
