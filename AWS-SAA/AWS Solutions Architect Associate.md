# AWS Solutions Architect - Associate

# AWS Overview

## AWS Global Infrastructure

- 24 Regions
    - Physical location in the world with 2 or more AZ
- 76 AZ
    - Discrete data centers with redundant power, networking and connectivity, housed in separate facilities
- 205 Edge Locations
    - Endpoints for AWS used for caching CloudFront content
- 11 Regional Edge Caches
    - Help reduce the load on CloudFront origin resources
    - Larger cache-width than any individual edge location so TTL can be longer

# **Identity Access Management (IAM)**

## Key Features of IAM

- Allows to manage users and their level of access to AWS console
- Centralized control of AWS account
- Global, does not apply to regions
- Shared access to AWS account
- Granular permission
- Identity Federation
    - Users can log-in via third-party credentials such as work credentials, Active Directory, Facebook, LinkedIn, etc...)
- Multifactor Authentication
- Temporary access for user/devices when necessary
- Set up custom password rotation policies

## Key Terminology for IAM

- Users
    - End users such as people and employees
    - Are created and exist within IAM service
    - Can login to Management Console
    - Can have long-term access keys
    - Can enable per-user MFA device
    - New users have no permissions at the beginning
    - New users are assigned an Access Key ID and a Secret Access Key on creation
- Groups
    - Collection of users that inhering a shared set of permissions
    - Can't be nested
- Policies
    - Determine authorizations
    - Written in JSON
    - Policy types
        - Managed Policy
            - AWS Managed
            - Customer Managed
        - Inline Policies
            - Policies written directly on IAM users, groups or roles
    - Can be created via
        - Generator
        - Hand written policies
    - Evaluation logic
        1. Default to implicit deny
        2. Look for explicit deny
        3. Look for explicit allow
    - What not to do
        - Embed credentials in
            - Code
            - ENV variables
        - Share with
            - Third parties
            - Hundreds of enterprise users
            - Millions of web users
- Roles
    - Use temporary credentials
    - Delegate permissions to
        - EC2 instance
        - AWS service
        - A user (elevate privileges)
        - Separate accounts (cross account access or sharing)
            - One you own
            - Third party
        - Roles for EC2 instances
            - Security Token Service provides temporary credentials that the EC2 hypervisor retrieves via the Instance Metadata Service
            - Credentials are rotated 5 minutes prior to expiration
        - Role for Cross Account Access
            - Accounts that need a trust relationship to access different resources owned by other accounts
            - Policy on the role to allow access to resources and on the user to be allowed to assume the role

## AWS New Account first steps

1. Enable CloudTrail (service that records all calls made to AWS API)
2. Create admin user in IAM
3. Enable MFA on root account
4. Enable Cost and Usage Report (detailed cost reports)
5. Log out of root account
6. Log back in as admin user
7. Create additional users/groups

## IAM Best Practices

- Root credentials
    - Email address + password
    - Protect at all costs
    - Do not use for day-to-day
- Follow principle of least privilege
- Rotate access keys
- Enable MFA
- Enable CloudTrail

# Simple Storage Service (S3)

## What is S3?

- Secure, durable, highly-scalable object storage
- Safe place to store files
- Data is spread across multiple devices and facilities
- Object-based ⇒ upload files into buckets
    - Between 0 bytes and 5 TB
- Unlimited storage

## S3 Basics

- Universal namespace
    - Bucket names must be globally unique
    - If created in US-West1, ***bucketname**.s3.amazonaws.com*
    - If outside, ***bucketname**.**region-code**.amazonaws.com*
- Objects
    - Key ⇒ name of the object
    - Value ⇒ data of the object
    - Version ID ⇒ important for versioning
    - Metadata ⇒ data about data you are storing
    - Subresources
        - Access Control Lists (ACLs)
        - Torrents
    - On successful upload, returns an HTTP 200 response
    - Not for operating systems or databases ⇒ block storage is more appropriate
- Data Consistency Model for S3
    - **Read after Write consistency** for PUTS
    - **Eventual consistency** for overwrite PUTS and DELETES
- Guarantees
    - Built for 99.99% availability
    - Amazon guarantees 99.9% availability
    - Amazon guarantees 99.99999999999% durability (11 9s)
- Features
    - Tiered Storage Available
    - Lifecycle Management
    - Versioning
    - Encryption
    - MFA for Deletion
    - Secure data using
        - Access control lists ⇒ fine grain object-level permissions
        - Bucket Policy ⇒ bucket-level access
- S3 Storage Classes
    - S3 Standard
        - 99.99% availability
        - 99.99999999999% durability
        - Stored redundantly across multiple devices in multiple facilities
        - Can sustain loss of 2 concurrent facilities
    - S3 - Infrequently Accessed (IA)
        - For data that is not accessed frequently but needs rapid access when needed
        - Lower fee compared to standard S3 but fee on retrieval
    - S3 - Intelligent Tiering
        - Automatically moves data to most cost-effective access tier with no performance impact
    - S3 OneZone - IA
        - For data that is not accessed frequently and does not require multi-AZ resilience
        - Even lower cost but fee on retrieval
    - S3 Glacier
        - Secure, durable and low cost storage for data archiving
        - Very cheap but fee on retrieval and retrieval can be extremely slow
    - S3 Glacier Deep Archive
        - Cheapest storage option
        - Retrieval time of 12 hours
- S3 charges
    - Storage
    - Requests
    - Storage Management Pricing ⇒ tiers
    - Data transfer pricing
    - Transfer Acceleration
        - Fast, easy and secure transfer of files over long distances between an S3 bucker and end users
        - Uses CloudFront edge locations
        - As data arrives to edge location it is routed to S3 over optimized network paths
    - Cross Region Replication
        - Automatically replicate buckets in different regions for redundancy or disaster recovery
- S3 pricing
    - S3 Standard
        - 50 TB / 450 TB / 500 TB increments
    - S3-IA
        - Constant price GB/Month
    - S3 Intelligent Tiering
        - 50 TB / 450 TB / 500 TB increments
    - S3 OneZone-IA
        - Constant price GB/Month
    - S3 Glacier
        - Constant price GB/Month
    - S3 Glacier Deep Archive
        - Constant price GB/Month

## S3 Security and Encryption

- Basics
    - All newly created buckets are private
    - Can setup access control via Bucket Policies and Access Control Lists
    - Can be configured to create access logs which log all requests made to the S3 bucket
    - These can be sent to another bucket or another account
- Encryptions
    - Encryption In Transit
        - SSL/TLS
    - Encryption At Rest (server side)
        - S3 Managed Keys (SSE-S3)
        - AWS Key Managed Service (SSE-KMS)
        - Server Side Encryption with Customer Provided Keys (SSE-C)
    - Encryption at Rest (client side)
        - Data is encrypted before being uploaded to S3

## S3 Versioning

- Stores all versions of an object, including writes and even deletions
- Uploading a new version adds up to the storage
- Total storage used is the total size of every version of every file
- Great backup tool
- Once enabled, it can only be suspended - not disabled
- Integrates with lifecycle rules
- When a new version of a file is uploaded, it defaults to private
- Can delete individual versions
- Can enable MFA Delete capability for additional security

## S3 Lifecycle Management

- Used to manage automated or deletion of stored objects or or transition to different storage classes through lifecycle rules
- Can use tag filters
- Can be used in conjunction with versioning with separate rules for current and old versions of files
- Can expire versions of objects
- Can clean incomplete multi-part uploads
- Rulesets are created and attached to buckets

## S3 Object Lock

- A method to store objects using the Write Once, Read Many (WORM) model.
- Prevents objects from being deleted or modified
- Can be for a specified amount of time or indefinitely
- Used for regulatory requirements or added security
- Retention periods
    - A fixed amount of time during which an object version is protected
    - Object retains timestamp in the metadata
    - After expiration object can be overwritten/deleted unless it has also a legal hold
- Legal hold
    - Prevents an object version from being overwritten/deleted without associated retention period
    - Remains in place until removed
    - Can be freely placed or removed by users who have `s3:PutObjectLegalHold` permission
- Multiple modes
    - Governance mode
        - Can't overwrite or delete an object version or alter its lock unless you have special permissions
        - Used to protect from deletion by most users while granting permissions to certain users to alter settings if needed
    - Compliance
        - Objects can't be deleted or overwritten by anyone, including root user
        - Retention mode and period can't be altered
        - Used for regulatory reasons

## Glacier Vault Lock

- Deploy and enforce compliance controls for individual S3 Glacier vaults
- Uses Vault lock policies that can't be changed once locked
- Can specify controls such as WORM in the policy