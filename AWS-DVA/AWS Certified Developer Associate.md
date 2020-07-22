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
        - Data can be stale → Updated in DB, old in cache
- Write Through → Write optimization strategy
    - Add or update cache when DB is updated
    - If requested data is already cached, it is a cache hit
    - When there is a write to DB, the write is replicated in the cache
    - Pros
        - Data is never stale
        - Write penalty instead of read penalty
    - Cons
        - Data is missing until updated/added in the DB → Combine with Lazy Loading
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
    - API Rate limits → Per-second limits on API calls, different from call to call
    - Service Quotas → Limit on how many resources we can run at the same time
    - AWS throttles requests if they reach the predetermined limits
    - If API calls are being throttled, use built-in exponential backoff to retry API calls
- CLI Credentials Provider Chain
    1. Command line options → `--region`, `--output`, `--profile`
    2. Environment variables → `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`
    3. CLI credentials → `aws configure` `~/.aws/credentials` 
    4. CLI configuration → `aws configure` `~/.aws/config`
    5. Container credentials → ECS tasks
    6. Instance profile credentials → EC2 Instance Profiles
- SDK Credentials Provider Chain
    1. Environment variables → `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`
    2. System properties
    3. Default credential profiles file → `~/.aws/credentials`
- API Request Signature (SigV4)
    - Protocol used to sign HTTP requests when not using SDK or CLI
    - Can use either HTTP Header options or Query Strings (ex: pre-signed S3 URLs)