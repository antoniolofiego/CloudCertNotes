# AWS Certified Developer Associate

# ElastiCache

## Cache considerations

- Data might be out of date but is eventually consistent, so caching is safe for certain type of datan
- Effective caching
    - Specific data patterns
    - Slow changing
    - Few frequently needed keys
    - Key-value data or aggregation results
- Ineffective caching
    - Anti-patterns
    - Fast changing
    - Many frequently needed keys

## Caching Patterns

- Lazy Loading (also called Cache-Aside or Lazy Population) → Read optimization strategy
    - Data is loaded in cache only when necessary
    - If requested data is already cached, it is a cache hit
    - If requested data is not cached yet, it is a cache miss, data is retrieved from DB and then written to cache (3 separate network calls)
    - Pros
        - Only requested data is cached
        - Resilient to node failures
    - Cons
        - Cache misses create delay for requests because of 3 calls
        - Data can be stale ⇒ Updated in DB, old in cache
- Write Through → Write optimization strategy
    - Add or update cache when DB is updated
    - If requested data is already cached, it is a cache hit
    - When there is a write to DB, the write is replicated in the cache
    - Pros
        - Data is never stale
        - Write penalty instead of read penalty
    - Cons
        - Data is missing until updated/added in the DB ⇒ Combine with Lazy Loading
        - A lot of data is always cached and it might be read very sparingly
- Adding Time To Live (TTL)
    - A cached item can be cleared manually, automatically when the memory is fully and the item has not been recently used or on a schedule (TTL)
    - TTL can range from seconds to days
    - Time should be sensible to the type of access patterns of the data that is being cached

# AWS CLI and SDK

## CLI Overview

- Command Line Interface for AWS services, it is a wrapper for the Python SDK (boto3)
- To use the CLI you need to generate access keys for the IAM user
- Access keys are downloadable only once and can't be retrieved if lost
- Credentials are stored in a file called `~/.aws`
- To use CLI on EC2, don't use personal credentials but create appropriate IAM Roles
- You can test whether you have permission to run certain commands without actually making API calls by specifying the `--dry-run` flag
- You can decode encoded auth failure errors by using the STS CLI tool
`aws sts decode-authorization-message --encoded-message <message-value>`
- An EC2 instance can access its metadata by curling `http://169.254.169.254/latest/meta-data`
- AWS Profiles
    - You can manage multiple IAM Accounts on the same CLI by using CLI Profiles
    - Run `aws configure --profile <profile-name>` to set up
    - Regular calls are made through the default profile
    - To use the other profiles, specify the `--profile` flag when making calls
- MFA on CLI
    - Needs a temporary session which is initiated with an STS `GetSessionToken` API call
    `aws sts get-session-token 
    --serial-number <arn-of-mfa-device> 
    --token-code <code> 
    --duration-seconds <seconds>`

## SDK Overview

- Available in many languages (Java, .NET, Node.js, PHP, Python, Go, Ruby, C++)
- SDK is used to issue programmatic API calls in code
- Uses default region `us-east-1`
- AWS Quotas/Limits
    - API Rate limits ⇒ Per-second limits on API calls, different from call to call
    - Service Quotas ⇒ Limit on how many resources we can run at the same time
    - AWS throttles requests if they reach the predetermined limits
    - If API calls are being throttled, use built-in exponential backoff to retry API calls
- CLI Credentials Provider Chain
    1. Command line options ⇒ `--region`, `--output`, `--profile`
    2. Environment variables ⇒ `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`
    3. CLI credentials ⇒ `aws configure` `~/.aws/credentials` 
    4. CLI configuration ⇒ `aws configure` `~/.aws/config`
    5. Container credentials ⇒ ECS tasks
    6. Instance profile credentials ⇒ EC2 Instance Profiles
- SDK Credentials Provider Chain
    1. Environment variables ⇒ `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`
    2. System properties
    3. Default credential profiles file ⇒ `~/.aws/credentials`
- API Request Signature (SigV4)
    - Protocol used to sign HTTP requests when not using SDK or CLI
    - Can use either HTTP Header options or Query Strings (ex: pre-signed S3 URLs)

# CloudFront

## CloudFront Caching

- Can cache based on headers, cookies, query string parameters
- Cache lives in the edge locations
- Static and Dynamic content should be cached separately in two distributions
- CloudFront has a Cache invalidation settings that can create rules on when to flush the caches

## CloudFront Signed URL/Cookies

- Used to provide controlled access to CloudFront distributed content
- Uses policies with URL expiration, allowed IP ranges, trusted signers
- Signed URL gives access to individual files, signed Cookies give access to multiple files

# Containerized Applications (ECS, ECR, Fargate)

## ECS Clusters

- Logical grouping of EC2 instances that run the ECS agent
- ECS agents register the instances as a part of the ECS Cluster
- These EC2 run specific AMIs that are made specifically for ECS

## ECS Tasks Definitions

- JSON-format metadata that tells ECS how to run a Docker container
- It lists
    - Image name
    - Port binding for container and host
    - Memory/CPU required
    - Environment variables
    - Networking information
    - Task roles ⇒ Optional IAM roles that the instances take on to interact with AWS services

## ECS Service

- Help define number of tasks that should run and how they should run
- Used to ensure that the desired number of tasks is running across the fleet
- Can use an Application Load Balancer with dynamic port forwarding to automatically send traffic to the correct ports on EC2 instances that are running more than one service

## Elastic Container Registry (ECR)

- Private Docker image repository
- You can host public ones on Docker Hub, but you might want to have some images private
- Access is controlled through IAM policies
- Can login to ECR via CLI ⇒ Must execute the output of the command, which is why `$( ... )`
CLI v1 ⇒ `$(aws ecr get-login --no-include-email --region <region>)`
- CLI v2 ⇒ `aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin 1234567890.dkr.ecr.<region>.amazonaws.com`
- Docker Push and Pull work via docker and are independent of the login

## Fargate

- Serverless version of ECS
- No need to manage infrastructure, handle EC2 creation and scaling, etc...
- Only need to create task definitions and AWS handles provisioning and deployment
- All the dynamic port mapping is handled by Fargate, but we have to specify the containter port

## ECS IAM Overview

- EC2 Instance Profile
    - Used by the ECS agent to make API calls to ECS
    - Used to send container logs to CloudWatch Logs
    - Used to pull Docker images from ECR
- ECS Task Role
    - Allows tasks to have specific roles
    - Can interact with AWS services
    - Different roles can be attached to different tasks running in the same EC2 instance

## ECS Task Placement

- When an EC2 Task is launched, ECS must determine where to place it within the CPU/RAM/Port constraints
- When an EC2 Task is removed (scale-in), ECS must determine which task to terminate
- You can define Task Placement Strategies and Constraints to help ECS make decisions
- Task Placement Process
    1. Identify instances that satisfy task definition requirements for CPU/RAM/Port
    2. Identify instances that satisfy task placement constraints
    3. Identify instances that satisfy task placement strategies
    4. Select instance for task placement
- ECS Task Placement Strategies ⇒ Best effort strategies
    - Binpack
        - Place tasks based on the least available amount of CPU or memory
        - Optimize usage, minimize number of instances in use
    - Random
        - Place tasks randomly
    - Spread
        - Place tasks evenly across specified values
        - Can be AZs, instances, etc...
    - Strategies can be combined together
- ECS Task Placement Constraints
    - distinctInstance
        - Place each task on a different instance
    - memberOf
        - Place task on instances that satisfy certain Cluster Query Language expressions

## ECS Auto Scaling

- CPU and RAM are tracked by CloudWatch so we can trigger auto scaling using metrics
- Different auto scaling strategies
    - Target Tracking ⇒ Target an average metric
    - Step Scaling ⇒ Scale based on CloudWatch Alarms
    - Scheduled Scaling ⇒ Scale based on predictable, scheduled changes
- ECS Auto Scaling is at the task level and not at the instance level
- Fargate makes it easier because it's serverless

## ECS Cluster Capacity Provider

- Used to auto scale ECS both at the service and at the instance level
- Determines the infrastructure that a task runs on and makes appropriate adjustments to fleet
- For ECS and Fargate, FARGATE and FARGATE_SPOT capacity providers are automatically added
- For ECS on EC2, the capacity provider has to be associated with an auto scaling group
- Custom capacity providers can be created and services can be built around them

# Elastic Beanstalk

## Elastic Beanstalk Overview

- Managed Platform as a Service used to deploy applications quickly and easily on AWS
- Built on top of all common services (EC2, ASG, ELB, RDS, etc...) leveraging CloudFormation
- Retain full control over configuration
- Free service, but pay for services provisioned
- All the infrastructure, deployment and configuration is handled by AWS
- Codebase is the only developer's responsibility
- Three components of Elastic Beanstalk are Application, Application version and Environment name
- An application version is deployed in an environment and they can be promoted to next env
- Can rollback to previous application versions
- Support for multiple languages
    - Go
    - Java SE/Tomcat
    - .NET
    - Node.js
    - Python
    - Ruby
    - PHP
    - Packer Builder
    - Single/Multicontainer/Preconfigured Docker
    - Custom community-backed platforms for other language (Uses AMIs and Packer)

## Elastic Beanstalk Deployment

- Deployment Modes
    - Single instance deployment ⇒ Best for development and testing
    - ASG + ELB ⇒ Best for production or pre-production environments
    - ASG only ⇒ Best for non-web app production (worker nodes, etc...)
- Deployment Options for Updates
    - All at once ⇒ Fastest way to update, but brief downtime as instances aren't available
    - Rolling ⇒ Update few instances at a time and move to another set of instances once done
    - Rolling with batch ⇒ Like rolling, but new instances are created to keep old app available
    - Immutable ⇒ New instances in new ASG with new version, swapping when healthy
    - Blue/Green Deployment
        - Not an out of the box feature of Elastic Beanstalk
        - Zero downtime, easier deployment
        - Create a new staging environment and deploy the new version there
        - The new environment can be validated independently from the original one
        - Use Route 53 weighted policies to redirect only portions of the traffic to the staging env
        - Swap URLs in Elastic Beanstalk when the testing is done

## Elastic Beanstalk CLI

- Elastic Beanstalk has its own CLI called `EB cli`
- Basic EB CLI commands are`eb create`, `eb status`, `eb health,` `eb deploy`, `eb logs`, `eb config` and `eb terminate`

## Elastic Beanstalk Lifecycle Policy

- You can store max 1000 app versions. If you reach 1000, you can't deploy new versions
- Lifecycle Policies phase out old versions based on time or storage
- Versions in use in any environment won't be deleted
- Can retain source bundle in S3 even if an application is deleted

## Elastic Beanstalk Extensions

- Files that encode specific parameter settings found in the console programmatically
- Files must have `.config` extension and in the `.ebextensions/` directory at root level
- YAML/JSON
- You can add multiple resources such as RDS, DynamoDB, etc... but these resources are deleted on environment deletion

## Elastic Beanstalk Cloning

- Clone environment with the same configuration
- Every resource type and configuration is copied but data is not preserved (ex: RDS)
- You can independently change settings on a cloned environment

## Elastic Beanstalk Migrations

- ELB and RDS need special care when used in Elastic Beanstalk
- ELB
    - Can't change the type of load balancer once it is picked
    - Steps for migration
        - Recreate a new environment configuration with no ELB
        - Select the correct ELB in the new environment
        - Swap traffic from the old environment to the new one
        - Terminate old environment
- RDS
    - You can provision an RDS database with Elastic Beanstalk but the DB will be tied to the environment lifecycle ⇒ Good for dev/test, bad for prod
    - Best approach is to create RDS independently and provide EB application with connection strings
    - Steps for decoupling RDS
        - Create an RDS snapshot
        - Protect RDS DB from deletion
        - Recreate new environment without RDS
        - Point your application to the existing RDS DB
        - Swap traffic from old environment to the new one
        - Terminate old environment and delete CloudFormation stack

## Docker in Elastic Beanstalk

- Single Docker
    - Provide `dockerfile` or `Dockerrun.aws.json v1` that specifies where a pre-built image is
    - Single Docker application does not use ECS but only Docker on an EC2 instance
- Multi Docker
    - Leverages ECS to run multiple container on various EC2 instances
    - Automatically provisions ECS Cluster, EC2 instances, Load Balancer and task execution
    - Requires `Dockerrun.aws.json v2` to generate ECS Task Definitions
    - Docker images must be pre-built and stored in ECR or Docker Hub

## HTTPS in Elastic Beanstalk

- SSL certificates must be loaded on the Load Balancer via console or using `.ebextensions/securelistener-alb.config`
- Certificates can be obtained by AWS Certificate Manager or CLI and security groups must allow traffic on port 443
- Can also redirect HTTP to HTTPs via ALB or via instance configuration (health checks should not be redirected)

## Difference between Web Server and Worker Environments

- Worker environments are for tasks that take long time to compute or are on a schedule
- These can be decoupled using SQS queues to send messages from web tier to worker tier

# CI/CD in AWS

## CI/CD Overview

- Continuous Integration
    - Allows developers to push tested code to a repository more often
    - Uploads code to a build server, tests it, builds it and lets you know if everything is okay
- Continuous Delivery
    - Reliable software releases whenever needed
    - Multiple releases every day
    - Automated deployment of passing builds

## CodeCommit

- Fully managed, highly available AWS version of GitHub
- Easy collaboration and fast code backup
- All CodeCommit repos are private and have no size limit
- All the code is in your AWS infrastructure
- Integration with Jenkins, Travis, CircleCI and other third-party CI tools
- Authentication is done through SSH (non-root accounts) or HTTPS with optional MFA
- Authorization is via IAM Policies/Roles
- Encrypted at rest with KMS, in transit through SSL
- CodeCommit Notifications
    - Via SNS/Lambda ⇒ Branch deletion, push to master, build system notification, code analysis
    - Via CloudWatch Events Rules ⇒ Pull request updates, commit events

## CodePipeline

- Visual continuous delivery workflow that orchestrates various steps of pipelines
- Orchestrates source code from CodeCommit/GitHub/S3
- Builds and tests using CodeBuild/Jenkins/other third party
- Deploys using CodeDeploy/Beanstalk/CloudFormation/ECS/other AWS services
- Works on stages, each made of parallel or sequential actions and can request manual approval
- Each stage generates artifacts, that are stored on S3 and passed to subsequent stages
- Stage changes trigger CloudWatch Events that can create SNS notifications (failed/successful pipelines, cancelled processes, etc)
- When a pipeline fails it stops and you can get additional info in the console
- If a pipeline can't perform an action, it is likely due to IAM Policy issues

## CodeBuild

- Fully managed build service ⇒ Alternative to Jenkins
- No servers to provision, scales indefinitely, no build queue
- Pay for actual usage time
- Runs on Docker for build reproducibility and can use custom Docker images
- Can run locally using Docker and CodeBuild Agent for troubleshooting
- Integrates with KMS, IAM, VPC and CloudTrail
- Source code comes from CodeCommit, CodePipeline, S3, GitHub, etc...
- Build instructions are specified in a `buildspec.yml` file in the root folder
    - Environment variables ⇒ Plain text or SSM Parameter Store encrypted
    - Phases
        - Install ⇒ Install dependencies
        - Pre-build ⇒ Run commands before build
        - Build ⇒ Actual build commands
        - Post-build ⇒ Run commands after build
    - Artifacts ⇒ What to upload to S3
    - Cache ⇒ What to store in S3 for faster future build processes
- Can cache artifacts or build files in S3
- Outputs an artifact to designated S3 artifacts bucket
- Logs are stored in S3/CloudWatch Logs
- Can use metrics with CloudWatch Alarms to trigger Events and send SNS notifications
- Supported Environments
    - Python
    - Java
    - Ruby
    - Go
    - Node.js
    - Android
    - .NET Core
    - PHP
    - Docker
- Access to service within CodeBuild pipelines
    - By default, CodeBuild containers are outside of your VPC, so they can't access VPC's resources
    - You have to specify a VPC configuration with proper Security Groups to access resources
    - Use cases ⇒ Integration tests, data manipulation/query, load balancing

## CodeDeploy

- Deploy applications automatically to multiple EC2 instances managed by the user
- Basically a managed alternative to Ansible/Terraform
- CodeDeploy Agent
    - Each EC2 instance must run the CodeDeploy agent that polls CodeDeploy continuously
    - To install the agent, in the instance shell you must run

    ```bash
    sudo yum update
    sudo yum install ruby
    sudo yum install wget

    cd /home/ec2-user
    wget https://<aws-codedeploy-region>.s3.<region>.amazonaws.com/latest/install
    chmod +x ./install
    sudo ./install auto
    sudo service codedeploy-agent start
    ```

    - You can tag instances to specify deployment environment
- CodeDeploy sends agents an `appspec.yml` file
    - Specifies deployment instructions
    - Sits at the root of the source code stored in GitHub or S3
    - The EC2 instance must have the appropriate role to access the files from S3
    - Structure
        - File ⇒ Where to get the source code
        - Hooks ⇒ Instructions on how to deploy the application
            - ApplicationStop ⇒ Stop previous application version
            - DownloadBundle ⇒ Source code from where it is stored
            - BeforeInstall ⇒ Actions and commands before installation
            - Install ⇒ Installation process
            - AfterInstall ⇒ Actions and commands after installation
            - ApplicationStart ⇒ Boot up applications and services
            - ValidateService ⇒ Specify health checks to make sure the app is correctly deployed
- Deployment setting
    - One at the time ⇒ One instance at a time, stops deployment if one fails
    - Half at the time
    - All at once ⇒ All together, fast but no healthy host (appropriate for dev)
    - Custom ⇒ Specify percentage of minimum healthy hosts
- Deploment Type
    - In-place ⇒ Applications are stopped, updated and restarted
    - Blue/Green ⇒ New revision is installed on a new set of instances and ELB is switched over
- CodeDeploy agents report failure/success of deployment on their instance
- Failed instances remain in "failed state" and future deployments deploy first to failed instances
- Can enable auto-rollback in case of failures
- Integration with CodePipeline ⇒ Can use its artifacts for deployment
- Can deploy to EC2 instances or on-prem, but on-prem has no Blue-Green deployment
- Deployments can target tagged EC2 instances, ASGs or a mix of both

## CodeStar

- Integrated solution that groups together all the CI/CD related services
- Centralized dashboard for all components
- Simple but not as customizable
- Free service, pay per provisioned services
- Can deploy to Lambda, EC2 and Elastic Beanstalk
- Supported Languages
    - Python
    - Java
    - Ruby
    - Go
    - Node.js
    - C#
    - PHP
    - HTML5
- Issue tracking integration with Jira and GitHub issues
- Integrates with Cloud9, AWS web-based IDE

# CloudFormation

## CloudFormation Overview

- Declarative way of describing AWS infrastructures for almost any resources
- Infrastructure as Code
    - No resources are created manually
    - Version control the entire infrastructure
    - Can review infrastructure changes in code and not via the console
    - Can optimize resource usage by creating and destroying stacks on schedule
- Cost tracking becomes easier as you can tag every resource and get stack estimates
- Can create as many stacks as needed to separate layers
- Template can't be updated but new ones have to be uploaded (S3 versioned in the background)
- Deleting a stack deletes all resources created with it
- Templates can be built in the console or using YAML/JSON files deployed via CLI
- All functions can take the form `Fn::<function>` or `!<function>`

## CloudFormation Resources

- Only mandatory field in a CloudFormation template
- Represent AWS services and resources
- A resource can reference other resources
- Resources have identifiers formatted as `AWS::<product-name>::<data-type>`

## CloudFormation Parameters

- Parameters are dynamic inputs provided to CloudFormation templates
- Used to reuse templates with different sets of inputs
- Referenced using `!Re`

## CloudFormation Mappings

- Mappings are fixed variables within CloudFormation templates
- Used to differentiate across environment, resource types, regions, etc...
- Used in case you know in advance all the possible values that something can take
- Accessed via `!FindInMap [ mapName, TopLevelKey, SecondLevelKey ]`

## CloudFormation Outputs

- Output values that can be referenced in other stacks
- Can be accessed via console or via CLI
- Allows for cross-stack interfacing to define portions of the stack in different templates
- You can't delete a stack if its outputs are referenced by other stacks
- Imported using `ImportValue`

## CloudFormation Conditionals

- Used to control creation of resources based on certain conditions
- Each condition can reference another condition, a parameter or a mapping
- Different set of conditions
    - `!And`
    - `!Equals`
    - `!If`
    - `!Not`
    - `!Or`

## CloudFormation Intrinsic Functions

- `!Ref` ⇒ Used to reference parameter values, resources ID,
- `!GetAtt` ⇒ Returns value for a specified attribute from a set
- `!FindInMap` ⇒ Return a named value from a key
- `!ImportValue` ⇒ Import output from another stack
- `!Join [ delimiter, [ comma-delimited list ] ]` ⇒ Join values with a delimiter
- `!Sub - String - ${ VarName: VarValue }` ⇒ Substitute values

## CloudFormation Rollback

- Default behavior after a stack creation failure is to delete everything that was created
- Can select to prevent rollback and just leave the stack in an incomplete state on failure
- If a stack update fails, the stack is rolled back to the previous known working state

## CloudFormation Nested Stacks

- Stacks that are parts of other stacks
- Used to isolate repeated components and reuse them across different stacks
- Cross stacks vs Nested Stacks
    - Cross stacks ⇒ Stacks have different lifecycles and are based on Outputs Exports
    - Nested stacks ⇒ Components are only part of the root stack

## CloudFormation StackSets

- Stack compositions that are created/updated/deleted all at once across regions/accounts
- Root account can create the StackSets, trusted accounts can create/update/delete stack instances from StackSets

# AWS Monitoring

## CloudWatch

- Provides metrics for every service in AWS
- CloudWatch Metrics
    - Metrics are variables grouped by namespace that can be monitored over time
    - Each metric has dimensions, which are attributes of metrics (instance ID, etc...)
    - Up to 10 dimensions per metric
    - EC2 Detailed Monitoring
        - EC2 instances by default deliver metrics every 5 minutes
        - With Detailed Monitoring you can get metrics every minute for an extra cost
        - Used for quicker ASG scaling prompts
        - 10 free Detailed Monitoring metrics in free tier
- CloudWatch Custom Metrics
    - User defined metrics
    - Can segment by dimensions
    - 1 minute resolution, up to 1 second resolution
    - Sent to CloudWatch via `PutMetricData` API Call
- CloudWatch Alarms
    - Used to trigger notification for metrics
    - Can be directed to SNS, EC2, ASG and other services
    - Alarms have different trigger quantities (min/max, percentages, etc...)
    - Metrics are evaluated in periods of seconds (only 10 or 30 seconds for custom metrics)
    - Multiple alarm states
        - OK
        - INSUFFICIENT_DATA
        - ALAM
- CloudWatch Logs
    - Can be sent to CloudWatch via SDK
    - Correct IAM Permissions are needed to send logs to CloudWatch
    - Can have expiration date
    - Many services can send logs natively (Elastic Beanstalk, ECS, Lambda, VPC Flow Logs, API Gateway, CloudTrail, Route53, etc...)
    - Can be analyzed via S3/Athena or ElasticSearch clusters
    - Log Architecture
        - Log groups ⇒ Arbitrary names representing application, KMS security at this level
        - Log streams ⇒ Instances of logs within applications/log files
    - Metric Filter
        - Filter expressions that look for specific occurrences in logs
        - Can be used to trigger alarms
        - They are not retroactive ⇒ Filtering happens only for data created after the filter
    - Encryption APIs
        - `associate-kms-group` ⇒ If the log group already exists
        - `create-log-group` ⇒ If the group does not exist, automatically associates KMS key
- CloudWatch Agents
    - Software that pushes logs to CloudWatch from EC2 instances/on-prem servers
    - Need the correct IAM Permission
    - Two versions of the agent
        - Logs agent ⇒ Old version, only send to CloudWatch Logs
        - Unified agent ⇒ New version, more system-level metrics (CPU, Disk metrics, RAM, network, processes), manage configuration via SSM Parameter Store
- CloudWatch Events
    - Events that are triggered either on a schedule or reacting to a service pattern
    - Trigger Lambdas. SNS/SQS/Kinesis messages
    - Returns a JSON file with information about the event

## EventBridge

- Upgraded version of CloudWatch events
- Advanced event busses for AWS that can be accessed by other AWS accounts
- Default Event Bus ⇒ from AWS services
- Partner Event Bus ⇒ from third party SaaS services or applications
- Custom Event Bus ⇒ from proprietary applications
- Schema Registry
    - Events in EventBridge can be analyzed to infer a schema
    - These schemas can be stored and versioned
    - Code can be generated for applications to correctly process the event data

## X-Ray

- Distributed application graphical debugger and analyzer built for microservices architecture
- Allows to troubleshoot performance, errors and service issues, track dependencies and visualize request behavior
- Compatible with Lambda, Beanstalk, ECS, ELB, API Gateway, EC2
- Works by tracing requests as they flow through the infrastructure with each service adding data
- Applications sends data in segments and sub-segments
- An end-to-end collection of segments is called a trace
- Traces can be annotated with indexed key-value pairs called annotations that can be filtered or non-indexed metadata that can't be filtered
- X-Ray SDK must be added to the code (Java, Python, Go, Node.js, .NET)
- X-Ray Daemon or X-Ray Integration must be installed/enabled on the service
    - `curl` on EC2 servers
    - `ebconfig` in Elastic Beanstalk or via console
    - Needs correct IAM permissions
- X-Ray sampling
    - Default sampling is first requests each second and 5% of every additional request
    - Can be adjusted to reduce the amount of requests that flow through X-Ray
    - Can adjust reservoir (minimum per second) and rate (percentage of additional requests)
    - Custom sampling does not require SDK changes as they are handled by the Daemon
- X-Ray APIs
    - `PutTraceSegments` ⇒ Uploads segment documents to X-Ray
    - `PutTelemetryRecords` ⇒ Used by X-Ray Daemon to upload telemetry
    - `GetSamplingRules` ⇒ Retrieve all sampling rules to know what data to send and when
    - `GetServiceGraph` ⇒ Returns main graph
    - `BatchGetTraces` ⇒ Retrieve all the traces specified by the ID
    - `GetTraceSummaries` ⇒ Get ID and annotations for traces in a specified time frame
    - `GetTraceGraph` ⇒ Service graph for one or more specific trace IDs
- X-Ray on ECS
    - ECS Cluster ⇒ One Daemon as a container in each instance
    - ECS Cluster Sidecar ⇒ One Daemon for each container in each instance
    - Fargate Sidecar ⇒ One Daemon for each container in each task

## CloudTrail

- Provides history of API calls made within AWS account by console, SDK, CLI or AWS services
- Enabled by default, used for governance, compliance and audit
- Logs from CloudTrail can be put in CloudWatch Logs
- If resources are deleted, look in CloudTrail first

# Simple Queue Service (SQS)

## SQS Overview

- A queue that works on a producer-consumer pattern to decouple applications
- Producers send messages to the queue via `SendMessage` API call
- Consumers poll the queue for messages, process them, and delete them via `DeleteMessage` API
- Consumers can scale in ASG by monitoring `ApproximateNumberOfMessages` CloudWatch metric
- Standard Queue
    - Unlimited throughput and unlimited messages in the queue
    - Message stays in the queue up to 14 days, default of 4
    - Low latency between publish and receive
    - 256KB maximum size of message
    - Delivers at least once (can have duplicates) and messages can be out of order (best effort)
- FIFO Queue
    - Messages are delivered in order and only once
    - Supports up to 300 API calls per second or 3000 with message batching
    - Currently there is no integration with SNS
    - Deduplicates messages in 5 minutes intervals, either by message content SHA-256 hash or specific deduplication ID
    - Can group messages using a `MessageGroupID` parameter, with one consumer per ID

## SQS Visibility Timeout

- When a message is polled by a consumer it becomes invisible to other consumers
- The time of invisibility is defined by the Visibility Timeout setting, default 30 seconds
- This is the time during which the consumer can process the messages
- If the consumer fails to process/delete the message in time it is made visible again in the queue
- Consumers can request more time by issuing calls to the `ChangeMessageVisibility` API

## SQS Dead Letter Queue

- If a message fails to be processed they are placed back in the queue
- If messages are placed back in the queue for too many times, they are put in a separate queue
- This queue can be used for debugging or other tasks
- Messages in the DLQ still do expire, so they need to be handled before expiration

## SQS Delay Queue

- Messages are delayed by a certain time before they are available to consumers
- Default is 0 seconds

## SQS Long Polling

- Consumers can wait for messages to be delivered in a queue if there are none available
- Reduces the amount of poll requests API calls and improves efficiency and latency as we ensure messages are read as soon as they are available in the queue
- Can be set between 0 and 20 seconds at the queue level or at the API call level

## SQS Extended Client

- Java library to use S3 as the repository for large data that we want to send via SQS
- Data is stored in S3 and metadata about the S3 location is sent as a message via SQS

## SQS API Calls

- `CreateQueue`
- `MessageRetentionPeriod`
- `DeleteQueue`
- `PurgeQueue` ⇒ Clear all messages in queue
- `SendMessage`
- `DelaySeconds` ⇒ Set delay time
- `ReceiveMessage`
- `DeleteMessage`
- `ReceiveMessageWaitTimeSeconds` ⇒ Specify long polling at API call level
- `ChangeMessageVisibility` ⇒ Change message visibility timeout
- Batch API calls for `SendMessage`, `DeleteMessage`, `ChangeMessageVisibilty`

# Simple Notification Service (SNS)

## SNS Overview

- Pub/Sub model for messaging and notifications
- A publisher publishes a message to a topic and all topic subscribers receive that message
- Can have really large numbers of subscribers to a topic (up to 10M) and up to 100K topics
- All subscribers receive the message unless there are filters in place
- Subscribers can be SQS,  Lambda, HTTP/S with retries, email, SMS or mobile push notifications
- Integration with multiple AWS services as SNS publishers

## Fan Out Pattern with SNS and SQS

- Used to publish the same message in multiple SQS queues
- Publish to an SNS topic and subscribe multiple queues to that topic
- Send message once, receive it multiple times
- SNS can't send messages to FIFO queues at the moment
- Use cases can be reacting to S3 events, as S3 can't send multiple messages by default

# Kinesis

## Kinesis Overview

- Managed real-time big data streaming alternative to Apache Kafka
- Data is replicated to 3 AZs
- Three separate products
    - Kinesis Stream ⇒ Streaming ingestion of real-time data at scale
    - Kinesis Firehose ⇒ Streaming data redirection tool
    - Kinesis Analytics ⇒ Real-time analytics on streams using SQL

## Kinesis Stream

- Divided in ordered shards, which are individual queues that retain data between 1 and 7 days
- Shards are provisioned, so billing is on a per-shard basis
- Can increase or decrease number of shards (resharding and merging)
- Producers publish data on one of the shards and consumers read data from either shard
- Data in shards can be processed multiple times during retention period but can't be deleted
- Multiple consumers can consume data from the same stream
- Producers write up to 1MB/s or 1000 messages/s per shard
- Consumers read up to 2 MB/s per shard
- Can batch send messages for increased efficiency
- Messages are ordered per shard
- Producers send data via SDK, consumers receive data via SKD or Kinesis Client Library (KCL)

## Sending Data to Kinesis

- `PutRecord` API call requires data and a partition key
- Messages are allocated among shards based on the hash of the partition key
- Same partition key always goes to the same shard
- Each message gets an always increasing sequence number once it is sent to a shard
- If the partition key is repeated often, certain shards are overloaded (hot partition problem)
- If one shard is overloaded, we get a `ProvisionedThroughputExceeded` exception

## Kinesis Client Library (KCL)

- Java Library that helps read records from a shard using distributed applications
- Each shard can be read by one KCL instance ⇒ 1:1 Shard/KCL ratio
- Progress is tracked via DynamoDB tables
- Can be deployed on EC2, Elastic Beanstalk or other

## SQS vs SNS vs Kinesis

- SQS
    - Consumer poll messages
    - Data is deleted after consumption
    - As many consumers as we want
    - Can delay individual message
    - No throughput provisioning
    - No ordering guarantees (except FIFO streams)
- SQS
    - Consumer subscribe to topics, messages are pushed
    - Up to 10M subscribers and 100k topics
    - Data is lost if not delivered
    - No throughput provisioning
    - Integrates with SQS for fan out architecture
- Kinesis
    - Consumer pull data
    - As many consumers as we want
    - Can replay data
    - Real-time big-data, ETL and analytics
    - Ordering at shard level
    - Temporary data retention
    - Must provision throughput

# AWS Lambda

## Lambda Overview

- Virtual serverless functions
- Up to 15 minutes of execution
- Run on demand
- Scales based on number of requests
- Multiple languages
    - Python
    - Node.js
    - Java
    - C# (.NET Core and Powershell)
    - Go
    - Ruby
    - Custom Runtime for community supported languages
- Key Integrations
    - API Gateway
    - DynamoDB
    - Kinesis
    - S3
    - CloudWatch Logs
    - CloudWatch Events/Event Bridge
    - CloudFront (Lambda@Edge)
    - SNS/SQS
    - Cognito
- Limitations
    - RAM ⇒ 128MB to 3,008MB in 64MB increments
    - Max execution time ⇒ 15 minutes
    - Environment Variables ⇒ 4KB
    - Disk capacity ⇒ 512MB
    - Concurrent executions ⇒ 1000
    - Compressed code size ⇒ 50MB
    - Uncompressed code size ⇒ 250MB (use `/tmp` for other files)

## Lambda Configuration

- RAM
    - 128MB to 3,008MB in 64MB increments
    - The more ram, the more vCPU credits
    - 1 full vCPU at 1792MB, so multithreading is necessary to benefit after that
    - If the function is CPU heavy, increase RAM to have more CPU
- Timeout
    - 15 minutes maximum running
    - 3 seconds default
- Lambda Layers
    - Allows creation of custom runtimes (like C++ or Rust)
    - Allows externalization of dependencies for reuse across different Lambdas
- Execution Context
    - Temporary runtime environment that initializes all dependencies
    - It is maintained for a while after execution for other Lambda invocations
    - Best practice is to initialize resources/connections outside of the handler
    - Can store temporary files or use disk space by leveraging the `/tmp` directory (Max 512MB)
    - Files in the `/tmp` directory are tied to the execution context and persist while it is frozen
- Dependencies
    - To use dependencies, install the packages alongside the code and zip it all together
- Deploying via CloudFormation
    - Inline ⇒ Write the function in the CF Template but can't use dependencies
    - S3 ⇒ Zip into S3 and refer to it in the CF template but on update, you have to update both S3 and CF
- Versioning
    - Can publish immutable versions of Lambda functions if we are ready to publish them
    - A version consists of the code and dependencies and can't be changed
    - Versions have increasing version numbers and get their own ARN
- Aliases
    - Versions can be referenced by Aliases ⇒ Pointers to specific versions of Lambda functions
    - Aliases are mutable, so you can change which version they are pointing to
    - Alias can be defined with any name, but sensible naming is helpful (dev, test, prod, etc...)
    - Aliases enable blue/green deployments by assigning weights to aliases
    - Aliases enable stable configuration of triggers/destinations as they can be used as targets
    - Aliases can't reference other aliases

## Lambda Synchronous Invocation

- Calls made via CLI, SDK, API Gateway, S3 Batch, CloudFront, ALB, etc...
- Results are returned immediately
- Errors are handled on the client side
- You can use API Gateway or ALB to let users invoke Lambda functions from HTTP
- When going through ALB, the Lambda has to be part of a target group
- HTTP requests are transformed into JSON and converted back to HTTP response by ALB
- ALB handles Multi-Header values via specific setting
- Synchronous Services
    - API Gateway
    - CloudFront (Lambda@Edge)
    - ALB
    - S3 Batch
    - Step Functions
    - Cognito
    - Kinesis Data Firehose
    - Others...

## Lambda Asynchronous Invocation

- Calls made via S3, SNS, CloudWatch Events
- Events are placed in an internal Event queue
- Lambda retries up to three times on errors
- In case of retries, it's important that the lambda function always returns the same value with the same inputs (idempotency) in order to catch them as duplicates in CloudWatch Logs
- DLQ for failed processing
- Asynchronous services
    - S3 Events
        - Can be enabled on most S3 events (`ObjectCreated`, `ObjectRemoved`, etc...)
        - Enabling versioning ensures event notification for each successful write
    - CloudWatch Events/EventBridge
        - Can be triggered on a schedule via CRON or Fixed Rate
        - EventBridge Rule for CodePipeline for triggering Lambda on State Changes
    - SNS
    - CloudWatch Logs
    - CodeCommit/CodePipeline
    - CloudFormation
    - Others

## Lambda Event Source Mapping

- Calls made via Kinesis Data Stream, SQS/SQS FIFO and DynamoDB Streams
- Records are polled from the source in all three cases
- The event source mapping polls the source and invokes the Lambda synchronously
- Event Streams (for DynamoDB Streams and Kinesis)
    - Iterator for each shard and processes items in order
    - One Lambda invocation per stream shard or up to 10 batches if using parallelization
    - Can read starting from different positions, either at specific timestamps or new/old items
    - Processed items are not deleted from the stream
    - Low traffic ⇒ Batch window to accumulate records
    - High traffic ⇒ Parallel multi-batch processing (10 batches per shard)
    - On error in a batch process, the entire batch is reprocessed until no error or expiration
    - Event source mapping can be configured to discard events, restrict retries or split batches
    - Discarded events go to destinations
- Queue (for SQS/SQS FIFO)
    - Queue is polled by event source mapping using long polling
    - Batch between 1 to 10 messages, up to 1000 batches per second
    - Queue visibility timeout should be set 6x the Lambda function timeout
    - Can use DLQ by associating one directly to SQS, not to the Lambda
    - Lambda scales up to process standard queues as quickly as possible, 60 per minute
    - Lambda scales up to the number of active message groups for FIFO queues (by GroupID)
    - If an error occurs, batches are returned to queue individually and might be reprocessed in different batches
    - Lambda might process the same item twice
    - Lambda deletes items from queues after successful processing

## Lambda Destinations

- Successful or failed event results can be sent to destinations
- Asynchronous invocations (both failed and successful events)
    - SQS
    - SNS
    - Lambda
    - EventBridge Bus
- Event Source Mapping (only for discarded events on Kinesis/DynamoDB)
    - SQS
    - SNS

## Using Lambda in a personal VPC

- By default, launched in an AWS-owned VPC so can't access your VPC's resources
- You can deploy it in your VPC by assigning it a VPC ID, security group and role
- Lambdas in a subnet, either private or public, don't have internet access or public IPs
- You need a NAT Gateway to give it access to the public internet
- DynamoDB can be accessed via IGW or via VPC Endpoint

## Lambda Concurrency

- Up to 1000 global concurrent executions
- Can set a limit to concurrent executions by setting a reserve concurrency at the function level
- Every call above the reserve concurrency triggers a throttle
- If there are no limits set, uptick in invocations from one Lambda can throttle all other Lambdas
- For asynchronous invocations, Lambda will put the request back in the queue automatically for up to 6 hours after a throttling error (429) or a system error (5xx) and reduce the retry interval
- Can allocate concurrency before invocation with provisioned concurrency to prevent cold start

## CodeDeploy automations

- Used to programmatically shift traffic among Lambda aliases
- Integrates with SAM templates
- Deployment strategies
    - Linear ⇒ Grow by X% every N minutes
    - Canary ⇒ Try X% then go to 100% after N minutes
    - AllAtOnce ⇒ 100% right away
- Rollback strategies are assigned using `PreTraffic` and `PostTraffic` hooks

## Lambda Best Practices

- Minimize handler work by moving heavy-duty tasks outside of it
- ENV variables for DB connections, passwords and similar (leverage KMS)
- Minimize deployment package size by using layers and breaking down functions
- Never use recursion in Lambdas

# DynamoDB

## DynamoDB Overview

- Fully managed, highly-available, distributed NoSQL datastore
- Millions of requests per second, trillions of rows, 100s of TB of storage
- Consistent low-latency performance
- IAM integration and event-driven programming using DynamoDB Streams
- Can migrate to DynamoDB from different DBs using DMS

## DynamoDB Table

- Each table has a Primary Key
- Made of items, can have infinite number of items but each item's max size is 400KB
- Each item has attributes, they can be null, can be added over time and can be nested
- Supports scalar, document and set data types
- Can be used for session state cache
- Can have Global Tables, that are multi-region, fully replicated, and high performance
- Concurrency Model
    - DynamoDB is Optimistic Locking database
    - Optimistic locking
        - Assumes that no conflict will occur
        - Data are read, the transaction processed, updates are issued, and then a check is made to see if conflict occurred
        - If a conflict occurred it is rolled back and repeated until successful
    - Items can be updated while being sure that it has not changed before further alterations
    - Writes can be conditional (based on previous conditions) or atomic (increase by x)
- TTL
    - Can define a per-row TTL to mark items for automated deletion after certain date/time
    - The TTL is a Number attribute with an UNIX epoch timestamp as expiration date
- Table Operations
    - Cleanup
        - Scan and Delete ⇒ Slow, expensive, inefficient
        - `DeleteTable` and recreate ⇒ Fast, cheap and efficient
    - Copy
        - Use AWS DataPipeline (leverages EMR) ⇒ Best solution
        - Create a backup and restore it in a new table ⇒ Might take some time
        - Scan and Write ⇒ Super inefficient

## Primary Key Types

- Partition key only (hash)
    - Unique for each item
    - Diverse so that data is distributed
- Partition key and Sort key
    - Combination must be unique
    - Data is grouped by partition key and sorted by sort key (also called range key)
- Choosing a Partition key
    - High cardinality
    - Maximizes distribution
- Data is divided in partitions based on the hash of the partition key
- Can calculate the number of partitions with the formula
$CEIL(MAX((RCU/3000+WCU/1000),(SIZE/10GB))$
- Can optimize partitions with Write Sharding, or adding random but predictable suffix to partition keys in order to spread writes across multiple partitions

## RCU and WCU

- Read and write capacities have to be provisioned per table
- Can set up auto-scaling of RCU/WCU but it's more expensive
- Throughput can be exceeded temporarily with burst credit but if you don't have any credit you get a `ProvisionedThroughputException` error
- These errors can be from hot keys, hot partition or very large items
- Reads can be either Strongly consistent or eventually consistent
- RCUs and WCUs are spread evenly among partitions
- Read Capacity Units (RCU)
    - 1 RCU = 1 strongly consistent read/second or 2 eventually consistent read/second for items up to 4KB
    - More KB, more RCUs
    - KBs are rounded up ⇒ 4.5KB item needs 2 WCUs
- Write Capacity Units (WCU)
    - 1 WCU = 1 write/second for an item up to 1KB
    - More KB, more WCUs
    - KBs are rounded up ⇒ 2.5KB item needs 3 WCUs

## Local Secondary Index (LSI)

- Up to 5 alternate range keys for a table, local to the hash key
- Must be exactly one scalar attribute, either String, Number or Binary
- Eventually consistent or strongly consistent reads
- Partition key must be the same as the base table
- Shares capacity with base table
- Defined at table creation time

## Global Secondary Index (GSI)

- Made of partition key and optional sort key
- Creates a new table where you can project attributes on
- Has its own provisioned capacity
- Can be modified or updated whenver
- Only eventually consistent reads
- If GSI's WCU are throttled, also the base table's writes will be throttled

## DynamoDB Streams

- A changelog stream of all CUD activities in DynamoDB
- Built on shards like Kinesis, but automatically provisioned by AWS
- Can be read by EC2/Lambda (as Event Source Mapping) to react in quasi-real-time to changes
- Needed for cross-region replication
- Streams data is retained for 24 hours
- Can choose what data is sent to the stream
    - KEYS_ONLY ⇒ Only key attributes of the item
    - NEW_IMAGE ⇒ Full item after modification
    - OLD_IMAGE ⇒ Full item before modification
    - NEW_AND_OLD_IMAGES ⇒ Full item both before and after modification

## DynamoDB Transactions

- All-or-nothing CUD operations on multiple rows in different tables at the same time
- Consumes twice the amount on RCU/WCU

## DynamoDB Accelerator (DAX)

- Seamless cache for DynamoDB
- All writes pass though DAX, microsecond latency for cached queries or reads
- Solves throttling issues for hot key reads
- 5 minutes TTL, multi-AZ, up to 10 nodes in the cluster
- Use in combination with ElastiCache if you cache data aggregation results

## DynamoDB APIs

- `PutItem` ⇒ Create or fully replace data
- `UpdateItem` ⇒ Partial replacement of data
- `DeleteItem` ⇒ Delete one row/item
- `DeleteTable` ⇒ Delete entire table
- `ConditionExpression` ⇒ Conditional write/update based on rules
- `BatchWriteItem` ⇒ Up to 25 `PutItem`/`DeleteItem` parallel calls (16MB max, 400KB per item)
- `GetItem` ⇒ Retrieve item based on primary key, eventually consistent by default but can set to strongly consistent
- `ProjectionExpression` ⇒ Specify which attributes to retrieve
- `BatchGetItem` ⇒ Up to 100 items at a time (16MB max)
- `Query` ⇒ Return item based on `PartitionKey`, optional `SortKey` range and client-side `FilterExpression` (Up to 1MB of data or number specified by a limit)
- CLI related flags
    - `--project-expression` ⇒ Attributes to retrieve
    - `--filter-expression` ⇒ Filter results once received
    - `--page-size` ⇒ Reduce page size for call optimization
    - `--max-items` ⇒ Maximum number of results returned by call, returns also `NextToken`
    - `--starting-token` ⇒ Which `NextToken` to use to resume reading

# API Gateway

## API Gateway Overview

- API service to manage REST APIs in a serverless architecture
- Supports WebSockets for real-time applications like chat, collaboration, multiplayer, etc...
- Handles environments, versions, security, API Keys, request throttling and validation, etc...
- Can import/export APIs using Swagger/OpenAPI
- Integrates with Lambda, HTTP, or any AWS Service
- Soft limit of 10,000 requests per second across all APIs per account (429 Error)
- Cache API responses
    - Default TTL is 300s, min 0s, max 3600s
    - Defined at stage level
    - Can override cache settings at the method level
    - Capacity between 0.5GB and 237GB
    - Client can invalidate cache using header `CacheControl:max-age=0` with proper IAM
- Multiple endpoint types
    - Edge-Optimized ⇒ Via CloudFront Edge locations
    - Regional ⇒ For users in one specific region
    - Private ⇒ Accessed within a VPC

## Integration Types

- Mock
    - Returns a response without connecting to the backend
    - Used for development and tests
- HTTP/AWS
    - Must configure integration request and response
    - Can use mapping templates for request and response
    - Mapping templates
        - Used to modify request and response
        - Rename/modify query string parameters, content of the body, headers
        - Uses Apache Velocity Template Language (VTL)
- Lambda Proxy
    - Request is the unmodified input to the Lambda
    - Can't use mappings for headers, query string parameters as they are passed as arguments
    - Lambda response is forwarded to the client
- HTTP Proxy
    - HTTP request is directly passed to the backend
    - Can't use mappings for headers, query string parameters as they are passed as arguments
    - HTTP response is forwarded to the client

## Stages

- For changes to be effective and accessible, APIs have to be deployed
- Changes are deployed to stages
- Each stage gets its own configuration parameters and can be rolled back as they are versioned
- Can deploy stages using Canary deployment
- Can set per-stage request limit to avoid account-level throttling
- Stage Variables
    - API Gateway equivalent of environment variables
    - Used to parametrize configuration values
    - Can be passed to Lambda/HTTP endpoints as context to configure specific parameters
    - Example: Stage variable that points to a Lambda alias for testing different versions
    - You need to add the correct resource-based policy via CLI to each Lambda alias

## Usage Plans

- Can define configurations for customers to be able to access APIs as service
- Users must supply API Key in the `x-api-key` header with their API calls
- Settings that can be modified
    - Who can access API stages and methods
    - How many calls they can make and how frequently
    - Use API Keys to meter connections and usage (generated by API Gateway or third party)
    - Throttling and quota limits
- Configuration steps
    1. Create APIs, configure methods to require API Keys and deploy to a Stage
    2. Generate or import API Keys to provide to customers
    3. Create usage plans with desired configuration
    4. Associate API Stage and API Keys with usage plan

## API Gateway Monitoring

- CloudWatch Logs at stage level
- X-Ray to enable tracing at request level in API Gateway and get full picture of API routes
- CloudWatch Metrics
    - `CacheHitCount` and `CacheMissCount` ⇒ Cache efficiency
    - `Count` ⇒ Total API requests
    - `IntegrationLatency` ⇒ Time between relaying a request to backend and receiving a response
    - `Latency` ⇒ Time between receiving request from client and return a response to the client
- Errors
    - `4XXError` (client-side) ⇒ 400 bad request, 403 access denied, 429 throttle
    - `5XXError` (server-side) ⇒ 502 bad gateway exception, 503 service unavailable, 504 integration failure (timeout)

## CORS

- Must be enabled to receive API calls from another domain
- Can be enabled via console
- Pre-flight request must contain three headers
    - Access-Control-Allow-Methods
    - Access-Control-Allow-Headers
    - Access-Control-Allow-Origin
- For Lambda Proxy, CORS must be enabled in the Lambda function return statement as its headers can't be modified due to being just relayed as-is

## API Gateway Security

- IAM based  ⇒ For users/roles already in your AWS account/organization
    - IAM Permissions
        - Policy authorization attached to User/Role
        - Authentication is via IAM, authorization is via IAM Policy
        - Uses SigV4 where IAM credentials are in the header of the requests
        - API Gateway does IAM policy check at request time
    - Resource Policies
        - Allow to set policies on the API Gateway to give specific users access to API gateway
        - Used for cross account access, VPC endpoints or whitelisting IP addresses
- Cognito User Pools ⇒ Already integrated with AWS, manage your user pool
    - Fully managed user lifecycle with automatic token expiration
    - Used for authentication, while authorization is handled by each API Gateway Methods
- Lambda Authorizer ⇒ For 3rd party tokens
    - Token-based authorizer (JWT, OAuth)
    - Returns ad-hoc IAM Principal and IAM Policy which is cached
    - Authentication is via 3rd party, authorization is via Lambda

# Step Functions

## Step Functions Overview

- Serverless workflow to orchestrate Lambda functions
- Support sequential calls, parallel calls, conditional calls, timeouts, error handling, etc...
- Integrates with EC2, ECS, API Gateway, SQS, on-prem servers
- Can require human approval steps
- Max execution of 1 year

## Workflows

- Standard workflow
    - 1 year duration
    - 2000 execution starts per second
    - 4000 state transitions per second
    - Pay per state transition
    - Execution can be debugged with CloudWatch Logs and Step Function console
    - Exactly-once execution
- Express workflow
    - 5 minutes duration
    - 100,000+ execution starts per second
    - Unlimited state transitions per second
    - Pay per execution, duration and memory consumption
    - Execution can be debugged with CloudWatch Logs
    - At-least-once execution

# Amazon Cognito

## Cognito Overview

- Used to provide users outside of your account access to interact with AWS-hosted applications
- Comprises Cognito User Pools, Cognito Identity Pools and Cognito Sync (which is deprecated)

## Cognito User Pools

- Serverless user database for web and mobile apps
- Handles login, password reset, email/phone verification, MFA, SAML/OpenID authentication
- Uses JWT as login output
- Can use a Hosted UI to handle your website's login
- Triggers ⇒ Custom Lambda functions that can be triggered at specific authentication stages
    - Auth events
        - Pre-authentication ⇒ Custom validation
        - Post-authentication ⇒ Event logging
        - Pre-Token generation ⇒ Custom token suppression
    - Sign-Up events
        - Pre-signup ⇒ Custom validation
        - Post-signup ⇒ Custom messages, analytics
        - Migrate user ⇒ From existing user directory to Cognito User Pools
    - Messages
        - Custom message ⇒ Localization or personalization
    - Token Creation
        - Pre-Token generation ⇒ Customize token attributes

## Cognito Identity Pools

- Used to provide temporary AWS credentials to access services in our accounts
- Identity providers include public ones (Facebook, etc...), Cognito User Pool, SAML/OpenID, custom third-party login servers or unauthenticated guest access
- Users can then access services via API Gateway or directly with Cognito-defined IAM policies

# Identity-related Additional Services

## Security Token Service (STS)

- Temporary (up to 1 hour) access to AWS resources
- STS APIs
    - `AssumeRole` ⇒ Role within account or cross account
    - `AssumeRoleWithSAML` ⇒ User logged in with SAML
    - `GetSessionToken` ⇒ For MFA
    - `GetFederationToken` ⇒ For federated users
    - `GetCallerIdentity` ⇒ Returns caller's IAM user/role detail
    - `DecodeAuthorizationMessage` ⇒ Decode error message when AWS API call is denied

## AWS Directory Services

- Centralized security/permission management, account creation tool using Active Directory
- AWS Managed Microsoft AD
    - Create AD in AWS, manage it locally, use MFA
    - Trust connection with on-prem AD
- AWS AD Connector
    - Proxy redirect to on-prem AD
    - Users can be managed only on on-prem AD
- AWS Simple AD
    - AD-compatible managed directory on AWS
    - Can't be joined with on-prem AD

# AWS Key Management Service (KMS)

## KMS Overview

- AWS managed keys to control access to data
- Full IAM integration for authorization
- Integrates with most AWS services
- Full management of keys (creation, enable/disable) and policies (rotation)
- Key usage is tracked by CloudTrail
- KMS API calls are billed at $0.03/10,000 calls
- KMS can encrypt up to 4KB of data per call
- Envelope Encryption
    - Used for encrypting data larger than 4KB
    - Uses `GenerateDataKey` KMS API Call
    - AWS offers Encryption SDK for Java, Python, C and JavaScript
    - SDK offers key caching to reduce API calls
- To access KMS, key policy must allow user and IAM policy must allow API calls
- KMS Keys are region-specific
- Key Policies
    - Control access to KMS keys
    - Can't control access without policies, so if there's no policy no one can access the key
    - Default key policy gives full access to root account
- KMS Quotas
    - KMS has a global quota for cryptographic operations per second
    - They are between 5,500 and 30,000 for symmetric keys and 500 or below for asymmetric
    - If quota is exceeded, requests are throttled
    - Can increase the quota via CLI/AWS Support or use Key caching to reduce API calls

## KMS Customer Master Keys

- Symmetric ⇒ AES-256
    - Single encryption key used for both decryption and encryption
    - AWS Services use symmetric keys
    - Used in envelope encryption
    - You will never have access to the unencrypted key but must use KMS API to use it
- Asymmetric ⇒ RSA and ECC key pairs
    - Public (encrypt) and private (decrypt) key pair
    - Used for encryption/decryption and for sign/verify operations
    - Public key can be downloaded, private key is not accessible
    - Used for encryption outside of the AWS environment when there's no access to KMS API
- CMK creation
    - AWS Managed ⇒ Service default, free
    - User keys created in KMS ⇒ $1/month
    - User keys imported (symmetric AES-256) ⇒ $1/month

## KMS API

- `Encrypt` ⇒ Encrypts up to 4KB of data
- `Decrypt` ⇒ Decrypts up to 4KB of data including data encryption keys (DEK)
- `GenerateDataKey` ⇒ Generates unique symmetric DEK returns plaintext copy of data key and encrypted version using a CMK that you specify
- `GenerateDataKeyWithoutPlaintext` ⇒ Used to generate a DEK to be used at a later time, must use `Decrypt` later

# Other security and encryption

## SSM Parameter Store

- Serverless secure storage of secrets and configurations
- Version control of secrets/configurations
- Integrates with KMS, CloudFormation, CloudWatch Events
- Parameter policies ⇒ TTL or notifications related to specific parameters
- Parameter tiers
    - Standard
        - 10,000 total number of parameters stored
        - 4KB max parameter size
        - No parameter-specific policies available
        - Free of charge
        - Standard throughput free, higher throughput (1000 req/second) $0.05x10,000 calls
    - Advanced
        - 100,000 total number of parameters stored
        - 8KB max parameter size
        - Parameter-specific policies available
        - $0.05 per parameter per month
        - Standard and higher throughput (1000 req/second) $0.05x10,000 calls

## AWS Secrets Manager

- Sole goal of storing secrets but very expensive
- Only supports encrypted secrets
- Can force rotation of secrets on schedule
- Currently supported secrets
    - Database credentials (RDS, Redshift, DocumentDB, etc...) ⇒ Username/Password
    - Other secrets (like API keys) ⇒ Key-value pairs

# Other Services

## AWS AppSync

- Managed GraphQL service
- Integrates with DynamoDB, ElasticSearch, Aurora and other services
- Retrieve real-time data using WebSockets
- Local data access and synchronization for mobile apps

## AWS SES

- Send email to SMTP interfaces and AWS SDK
- Receive email with integrations with S3, SNS and Lambda

## AWS Certificate Manager

- Host public SSL certificates in AWS, either AWS provided or third-party provided
- Integrates with ELB, CloudFront, API Gateway
