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