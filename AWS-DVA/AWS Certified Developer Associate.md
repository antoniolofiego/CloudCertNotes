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