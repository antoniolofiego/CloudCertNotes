# AWS Certified Developer Associate

# ElastiCache

## Cache considerations

- Data might be out of date but is eventually consistent, so caching is safe for certain type of data
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