# AWS Solutions Architect - Associate

# AWS Global Infrastructure

## Overview

- 24 Regions
    - Physical location in the world with 2 to 6 AZs, usually 3
    - Most AWS services are region-scoped
- 76 AZ
    - Discrete data centers with redundant power, networking and connectivity
    - Housed in separate facilities and isolated from disasters
    - Connected with high bandwidth, low latency networking
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
- Multifactor Authentication can be set up
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
- Roles
    - Use temporary credentials
    - Internal usage
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
- Policies
    - Determine authorizations
    - Written in JSON
    - Are attached to users, groups or roles
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

## AWS New Account first steps

1. Enable CloudTrail (service that records all calls made to AWS API)
2. Create admin user in IAM
3. Enable MFA on root account
4. Enable Cost and Usage Report (detailed cost reports)
5. Log out of root account
6. Log back in as admin user
7. Create additional users/groups

## IAM Federation

- For large enterprises
- Used to integrate your own repository of users with IAM
- Can login into AWS using company credentials
- Uses SAML standard (Eg: Active Directory)

## IAM Best Practices

- Root credentials
    - Email address + password
    - Protect at all costs
    - Do not use for day-to-day
- Follow principle of least privilege ⇒ Provide users with the minimal amount of permissions to perform their job
- One IAM User per physical person
- One IAM Role per application
- Rotate access keys
- Enable MFA
- Enable CloudTrail
- What not to do
    - Embed credentials in
        - Code
        - ENV variables
    - Share with
        - Third parties
        - Hundreds of enterprise users
        - Millions of web users

# Elastic Compute Cloud (EC2)

## What is EC2?

- Most popular AWS offering
- Allows you to launch instances of virtual machines

## Methods of connection to EC2 instances

- SSH ⇒ Mac, Linux, Windows 10
- Putty ⇒ Windows ≤ 10
- EC2 Instance Connect ⇒ All platforms
    - Browser-based SSH connections
    - Creates a temporary key that lets you log in without the secret key
    - Only works on Amazon Linux 2 AMIs
    - Does not work if SSH Port 22 is deleted

## Security Groups

- Firewalls at the instance level
- Outside of instances, so you can't check unathorized inbound from inside an instance
- Control inbound and outbound traffic of EC2 instances
- Sets controls on allow, inbound and outbound ports
- Defaults to inbound traffic blocked and outbound traffic authorized
- Act on
    - Port access
    - Authorized IP ranges (IPv4, IPv6)
    - Inbound/outbound network
- Can be attached to multiple instances
- Locked to region/VPC combinations
- If application times out it is an security group issue
- If application returns a "connection error" it's an app-level error or instance error (Not launched/terminated/stopped)
- Can reference other security groups for load-balancing purposes

## Private, Public and Elastic IPs

- Two types of IPs in networking
    - IPv4 ⇒ 122.168.1.1
        - Most common format
        - 2^32 unique addresses (~3.7 billion)
    - IPV6 ⇒ 2001 : 0db8 : 85a3 : 0000 : 0000 : 8a2e : 0370 : 7334
        - Most recent version
        - 2^128 unique addresses (~340 undecillion)
        - Solves IoT issues
- Public IP
    - Machine can be identified on the network
    - Has to be unique globally
    - Can be easily geolocated
- Private IP
    - Machine can be identified on the private network only
    - Has to be unique in the network
    - Two different private neworks can have the same IPs
- Elastic IP
    - A public IP for an instance that does not change when the instance is stopped
    - It is a public IPv4 address that you own as long as you don't delete it
    - Can be attached only to one instance at a time
    - Can be remapped to another instance in case of an instance or software failure
    - Up to 5 Elastic IPs per account (soft limit)
    - Avoid using Elastic IPs as they are usually a sign of a poor architectural decision somewhere
- Default EC2 configuration
    - A private IP for AWS network ⇒ can't use this to SSH in the machine because we are not in the same network
    - A public IP for the WWW

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

## S3 Performance Optimization

- S3 prefixes
    - The pathway between bucket name and object's key
    - S3 has extremely low latency, 1st byte out within 100-200 ms
        - High number of requests per second per prefix
            - 3,500 PUT/COPY/POST/DELETE
            - 5,500 GET/HEAD
- Optimization strategies
    - Spreading reads across prefixes
        - More prefixes means more requests per seconds that can be handled
        - Limitations with KMS
            - Separate KMS API calls for upload and downloads
                - Upload ⇒ `GenerateDataKey`
                - Download ⇒ `Decrypt`
            - The API has hard limits with requests per second (KMS Quota)
                - Region specific, but either 5,500, 10,000 or 30,000 requests per second

    - Multipart Uploads
        - For files over 100MB
        - Required for files over 5GB
        - Allows for upload parellelization
    - S3 Byte-Range Fetches
        - Parallelize downloads by specifying byte ranges
        - If download fails, it fails only in certain byte ranges
        - Can be used to download partial files

## S3 Select

- Use SQL expressions to retrieve subset of data from objects
- Performance boost of up to 400% and up to 80% cheaper than manipulating the full object

## Glacier Select

- Run SQL queries against Glacier directly
- Used by companies in highly regulated industries where companies write data directly to Glacier for compliance purposes

## Cross-Account S3 Bucket Sharing with AWS Organizations

- Three ways to share S3 buckets across accounts
    - Bucket Policies and IAM roles (across entire bucket) ⇒ Programmatic access only
    - Bucket ACLs and IAM roles (individual objects) ⇒ Programmatic access only
    - Cross-Account IAM roles ⇒ Programmating and console access

## Cross-Region Replication (CRR)

- Used to copy objects across Amazon S3 buckets in different AWS Regions
- Requires versioning enabled in the bucket that we are trying to replicate
- Used for
    - Meeting compliance requirements
        - Some compliance requirements dictate data storage to be in very distant locations
    - Minimize latency
        - If end users are in two geographic locations, they can access objects faster if there is a copy in a nearby region
    - Increased operational efficiency
        - Separate compute clusters in different regions can operate more efficiently on locally available data copies
- On creation of the replication rule, we can change storage class for replicated objects and we can change object ownership
- New buckets inherit the permission from souce bucket
- Objects are already in the bucket are not automatically replicated to the target bucket, so they have to be uploaded manually
- File modifications in one bucket are replicated to the other bucket
- Delete markers and individual version deletions are not replicated across buckets

## Transfer Acceleration

- Uses CloudFront Edge Network to accelerate uploads to S3
- Can use distinct URL to upload directly to edge location
    - `bucketname.s3-accelerate.amazonaws.com`

## DataSync

- Allows to move large amount of data to AWS
- Used on-premise
- Install an AWS DataSync agent on your server and connect to the NAS to copy data to/from AWS
- Automatically encrypts data and accelerates transfer over the WAS
- Automatic data integrity checks in-transit and at-rest
- In the AWS region, DataSync securely connects to S3, EFS or FSx for Windows File Server to copy data and metadata