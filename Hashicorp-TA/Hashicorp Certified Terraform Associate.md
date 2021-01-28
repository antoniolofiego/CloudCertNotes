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