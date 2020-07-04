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
- SSH connection ⇒ `ssh -i path/to/your/private/key ec2-user@public-ip`

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
    - To allow HTTP access, an HTTP rule to allow all inboud traffic on port 80 has to be added to the security group

## EC2 User Data

- A script that can be attached to an instance when the machine starts
- It only runs on the instance's first start with the root user ⇒ No need for `sudo su`
- Used to automate boot tasks such as
    - Updating the OS
    - Installing packages
- Need a #!/bin/bash shebang at the top of the script

## EC2 Instance Launch Types

- **On-Demand (least commitment)**
    - Default pricing structure for EC2 instances
    - No upfront cost and no commitment
    - Pay per hour (forward hour) or per second (after first minute) depending on instance
    - Best for short-term, spikey or unpredictable workloads and for testing/experimenting
- **Reserved (best long-term)**
    - For apps that have steady-state, predictable usage or need reserved capacity (eg: databases)
    - Price is dependent on
        - Term: length of commitment (1 or 3 year)
        - Payment option: all upfront, partial upfront or no upfront
        - Class offering:
            - Standard ⇒ up to 75% cost reduction but can't change instance types
            - Convertible ⇒ up to 54% and can change instance type to greater or equal in value
            - Scheduled ⇒ used for specific time windows and for fraction of day/week/month
    - Unused RIs can be sold in the RI Marketplace
    - RIs can be shared between multiple accounts within an organization.
- **Spot (biggest savings)**
    - Spot instances provide up to 90% discount compared to on-demand
    - AWS has unused compute capacity that it can sell to maximize utility of unused servers
    - It is sold at a spot price that is defined by supply/demand
    - You bid for a maximum price you want to pay
    - Can be terminated at any time if the computing is needed by an on-demand customer and your max price is less than the current spot price
    - Charged on hourly base, if your instance is terminated by AWS you don't get charged for partial hours.
    - If spot price gets above your max price, you have a 2-minute grace period where you can choose to either stop or terminate the instance
        - Stop if the workload needs to be resumed and the state of the machine is important
        - Terminate if the workload is stateless and can be restarted with no problem
    - Spot block
        - Reserve the spot for a specified time frame without interruptions
        - In extremely rare cases this might be reclaimed anyways
    - Most useful for applications that are feasible only at very low computing costs
        - Batch processing
        - Image processing
        - Flexible workloads
        - Big data workloads
    - Best combo ⇒ reserved instance for baseline + spot instances/on-demand for peaks
    - Spot requests
        - Need to specify
            - Max price
            - Number of instances
            - Launch specification ⇒ AMI
            - Validity period
            - Request type
                - One time
                    - Request is fullfilled ⇒ Intance is launched ⇒ request is completed and closed
                - Persistent
                    - If instances are terminated requests are recreated until the validity period ends
                    - To fully cancel a persistent request, cancel the request first and then terminate the instances otherwise the request gets fulfilled again
        - If a request is canceled it does not automatically terminate the instances that were launched at earlier stages with it
- **Dedicated (most expensive)**
    - Used to meet regulatory requirements of server-bound licensing that does not support multitenancy or cloud deployment
    - 3-year period reservations
    - Basically like buying a dedicated hardware and you are physically separated from other customers
    - Can't share across accounts

## EC2 Instance Types

- A good mnemonics is from ACloudGuru ⇒ FIGHT DR MC-PXZ (I pronounce it Fight Doctor McPixie)
- F ⇒ Field Programmable Gate Array (FPGA)
    - Big data, real-time video processing, financial analytics, etc...
- I ⇒ High Speed Storage
    - NoSQL, databases, data warehousing
- G ⇒ Graphics Intensive
    - Video encoding, 3D streaming
- H ⇒ High throughput
    - MapReduce, HDFS distributed file systems
- T ⇒ Burstable general purpose
    - General OK performance that adapt CPU to the needs of the process
    - Comes with burst credits that accumulate over time
        - Burstable
            - CPU increases performance for spikes
            - Uses a burst credit each burst
            - If it bursts when out of credits, CPU performance drops to minimum
        - Unlimited
            - Unlimited CPU performance bursts
            - More expensive
    - Good for Web servers and databases with unexpected/spiky traffic
- D ⇒ Dense storage
    - Hadoop, file servers
- R ⇒ Memory optimized
    - Memory intensive apps
- M ⇒ General purpose
    - App servers
- C ⇒ Compute optimized
    - CPU intensive apps
- P ⇒ Graphics and general purpose GPU
    - Machine learning, deep learning, bitcoin mining
- X ⇒ Memory optimized
    - SAP Hana, Spark
- Z ⇒ Sustained all-core frequency
    - Electronic Design Automation (EDA), gaming, database workloads with high per-core licensing costs

## EC2 Spot fleet

- Combination of Spot instances and optional On-Demand instances
- The Spot Fleet tries to meet targets within price constraints
- Allows to dynamically request the most cost effective Spot instance
- Spot fleet requests can have multiple launch pools
- Stops launching instances when reaching either target capacity or max price
- Launch pool
    - Set of definitions of instances
    - Includes
        - Type
        - AMI
        - AZ
- Spot Fleet strategies
    - Lowest Price
        - Lowest price pool
        - Best for short workloads and cost optimization
    - Diversified
        - Multiple pools
        - Best for consistent availability and long workloads
    - Capacity Optimized
        - Pool for optimal capacity for instance number

## EC2 AMI

- A required base image for the launch of an instance
- Can be customized with instructions on specific settings and software
- Can be use to launch multiple instances with the same configuration
- Region-specific
- Benefits of AMI
    - Pre-installed packages and software
    - Faster boot ⇒ faster than using User data
    - Better security
    - Known configurations
    - Changes can be done in one place instead of reconfiguring multiple instances
    - Many pre-made public free or rentable AMIs optimized for specific softwares/workloads
- AMIs are stored in S3 and are by default private and locked to account/region
- AMIs can be shared with other AWS accounts by modifying the image permission, but ownership does not change when it is shared
- Copying an AMIs that has been shared makes you the owner of that AMI
- You can't directly copy encrypted AMIs or AMIs that have an associated `billingProduct` code
    - Encrypted ⇒ If you have the underlying encryption key, you can use it to re-encrypt it with a key of your own
    - billingProduct ⇒ Can only copy from a running instance

## EC2 Placement Groups

- Custom EC2 placement strategies
- Different strategies available
    - Cluster
        - Instances are in a low latency group in a single rack and a single AZ
        - If a rack fails, all instances fail
        - Good for low-latency, high network throughput dependent applications
    - Spread
        - Instances are spread across different hardware in different AZs (max 7 instances per AZ)
        - Reduced risk for simultaneous failure
        - Good for critical, high-available applications
    - Partition
        - Instances are spread across different partitions and racks within an AZ
        - Partitions are failure-isolated, max 7 per AZ
        - Scales to 100s of instances per group
        - Instances access partition information as metadata
        - Good for Hadoop, Kafka, Cassandra, HDFS

## Elastic Network Interfaces (ENI)

- Representation of a Virtual Network Card in a VPC through a logical component
- Linked to a specific AZ
- Each ENI has different attributes
    - Primary private IPv4 IP address
    - One or more secondary IPv4 IP addresses
    - One Elastic IP (IPv4) per private IP
    - One Public IPv4 IP
    - A MAC address
    - One or more security groups
- ENIs are independent of instances and can be attached dynamically to EC2 instances for failover

## EC2 Hibernate

- Preserves in-memory state
- Instance boot is extremely fast
- It works by making a copy of the RAM state to the root EBS volume
- Requirements
    - Root volume has to be EBS encrypted and large enough to store the RAM dump
    - C, M and R instance families (3 to 5)
    - RAM < 150 GB
    - Not supported for bare metal instances
    - Amazon Linux 2, Linux, Ubuntu and Windows AMIs
    - On-Demand and Reserved instances
    - Can't hibernate longer than 60 days

## Exam questions related to EC2 for Solutions Architect

- EC2 Instances are billed by the second, t2.micro is free tier
- SSH on Linux/Mac, PuTTY on Windows
- SSH on Port 22, lock down security group to known IPs
- Timeout issue ⇒ security group issue
- Permission issue ⇒ `chmod 0400` issue
- Security groups can target other security groups or IPs
- Difference between Public, Private and Elastic IP
- Use EC2 User Data to customize the first boot of an instance
- EC2 Launch Types
- EC2 Instance families
- Custom AMIs (region scoped, but can be copied) ⇒ faster boots
- EC2 Placement groups ⇒ cluster, spread, partition
- EC2 Hibernate

# Elastic Block Storage (EBS)

## EBS Overview

- Network drive used for persistent storage of EC2 data
- Independent from EC2 instance but can be attached to one
- AZ scoped
- Billed per provisioned capacity (GB or IOPS)
- Can scale the size over time

## EBS Volume Types

- Each is characterized in size, IOPS and throughput
- Boot volumes
    - GP2 (SSD) ⇒ General purpose
        - Recommended for most workloads
            - Virtual desktops
            - Low-latency apps
            - Dev/test environments
        - 1GB - 16TB
        - Small GP2s can burst to 3000 IOPS
        - Max IOPS 16,000 for volume sizes greater than 5,334GB
        - Below 5,334 we have up to 3 IOPS/GB
    - IO1 (SSD) ⇒ High performance
        - For critical business applications with sustained IOPS performance or more than 16,000 IOPS
        - Large databases (MySQL, MongoDB, Cassandra,...)
        - 4GB -16TB
        - Provisioned IOPS 100 to 64,000 (for Nitro) or 32,000 (for non-Nitro instances)
        - Max IOPS to volume ratio is 50:1
- Non-Boot volumes
    - ST1 (HDD) ⇒ Low-cost, frequently accessed high throughput
        - For streaming workloads with fast throughput at low costs
        - Big data, data warehouses (Apache Kafka)
        - 500GB - 16TB
        - Max IOPS is 500
        - Max throughput is 500 MB/s and it can burst
    - SC1 (HDD) ⇒ Low-cost, infrequently accessed
        - Throughput optimized storage for large, infrequently accessed volumes of data
        - Optimal if goal is low cost
        - 500GB - 16TB
        - Max IOPS is 250
        - Max throughput is 250MB/s and it can burst

## EBS Encryption

- On EBS creation
    - Data at rest is encrypted
    - Data In flight is encrypted
    - Volumes created from (un)encrypted snapshots are (un)encrypted
    - Snapshots of (un)encrypted volumes are (un)encrypted
    - Copies of unencrypted snapshot can be encrypted
- Low impact on latency
- Automated
- Uses KMS (AES-256)
- Steps to encrypt unencrypted EBS Volume
    1. Create snapshot
    2. Encrypt the snapshot (copying it will automatically encrypt it)
    3. Create an EBS from the snapshot (will inherit encryption)

## EBS Snapshots

- Incremental ⇒ backups only changed blocks
- EBS backup should not be used while there is a lot of traffic because it consumes IO
- Stored in S3 but hidden
- Recommended, but not necessary, to detach the volume to take a snapshot
- Max 10,000 snapshots
- Can copy snapshots across AZs/Regions
- Can make AMI from snapshot
- Snapshot taking can be automated with AWS Data Lifecycle Manager
- EBS volumes restored by snapshot need to be pre-warmed (`fio` or `dd` to read full volumes)

## EBS Migration

- Steps to migration
    1. Snapshot the volume
    2. Optionally copy the volume to a different region
    3. Create a volume from a snapshot in the AZ you want

## EBS RAID

- Used for increasing IOPS or to mirror EBS volumes
- Support is OS dependent
- RAID 0 and 1 are most common options
    - RAID 0 ⇒ Increased performance
        - Combine two or more volumes and get combined disk space and I/O
        - If any disk fail, all data fails
        - Good for high IOPS tasks without fault-tolerance or already replicated DBs
    - RAID 1 ⇒ Increase fault tolerance
        - Mirror data to two or more volumes
        - If any disk fail, data will persist in others
        - Uses x times the network, where x is the number of volumes in the RAID
        - Good for fault-tolerance dependent applications
- RAID 5, 6 and 10 are not recommended

## Instance Store (ephemeral storage)

- Some instances use Instance stores instead of EBS volumes
- Instance store is block storage that is destroyed when the machine is stopped or terminated
- Physically attached to a machine
- Significantly high IOPS performance because they are physical (up to 2 million+)
- Good for buffer/cache data
- Data persists reboot
- Can't be resized or backed up automatically
- Risk of data loss if hardware fails ⇒ Replicate across instances if data is sensible/important
- Up to 7TB but can be stripped up to 30TB

# Elastic File System (EFS)

## EFS Overview

- Fully managed network file system that can be mounted on many EC2 instances in many AZ
- Highly available and scales automatically, but very expensive and on a pay-per-use basis ⇒ No capacity planning
- Uses NFSv4.1 protocol
- Attach a security group to the EFS to accept connections from EC2 instances
- All EC2 instances that are connected share the same files
- Useful for content management, data sharing and web servers
- Only on Linux AMIs ⇒ Uses POSIX FS with standard API
- Can be encrypted by KMS

## EFS Performance and Storage Classes

- EFS scale
    - 1000s of concurrent NFS clients with 10GB/s+ throughput
    - Can grow to PB-scale NFS automatically
- EFS performance modes
    - General purpose ⇒ Latency optimized (web served, CMS)
    - Max I/O ⇒ Higher latency, higher throughput, higly parallel (Big data, etc..)
- EFS storage tiers ⇒ Can move around with Lifecycle Management
    - Standard: regular frequently accessed files
    - Infrequent Access (EFS-IA): fee on file retrieval, lower storage costs

## EBS, EFS and Instance Store for Solution Architect Exam

- EBS
    - Can be attached to one instance at a time
    - Locked at the AZ level
    - GP2 ⇒ IO increases as disk size increases
    - IO1 ⇒ IO can increase independently of disk size
    - Migration to different AZ works via snapshot restoration
    - Root EBS volumes get terminated by default if instance is terminated
- EFS
    - Can be attached to 100s of instances across AZ
    - Files are shared across instances
    - Only for Linux
    - More expensive than EBS
    - Can cost-manage via Lifecycle management and EFS-IA
- Instance Store
    - Physical storage attached to EC2 instance
    - Files are lost on stop or termination
    - Faster I/O because of physical drive
    - Data persists on reboot
    - Can't be resized or automatically backupped

# Simple Storage Service (S3)

## S3 Overview

- Secure, durable, highly-scalable object storage
- Safe place to store files
- Data is spread across multiple devices and facilities
- Object-based ⇒ upload files into buckets
    - Between 0 bytes and 5 TB
- Unlimited storage

## S3 Basic Concepts

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
- Replicated data is read only

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

# Elastic Load Balancer (ELB)

## Scalability Overview

- Scalability means that a system can handle greater loads by adapting
- Scalability comes in two flavors
    - Vertical scalability
        - Increasing compute capacity (size of instance)
        - Good for non-distributed servers like RDS, ElastiCache
    - Horizontal scalability ⇒ Elasticity
        - Increasing number of servers ⇒ Scale out, scale in
        - Implies distributed systems
        - Used for Load Balancers and Auto Scaling Groups
        - Good for modern web applications
- Scalability has ties to high availability but they are two different concepts
    - HA means having fault tolerance to the loss of data centers
    - Achieved by running the same instances across multiple AZs
    - Can be active (I deploy multiple EC2 instances in different AZs) or passive (I enable multi-AZ deployment in RDS)
    - User for multi-AZ ASG and multi-AZ Load Balancer

## Load Balancing Overview

- Servers that direct internet traffic to and from multiple EC2 instances and spreads the load
- End users interface with one DNS endpoint, the Load Balancer, while the backend can be made of many instances
- Performs health checks of instances to adaptively handle point of failures
- Provides SSL termination for websites (HTTPS)
- High availability across AZs
- Enforce cookie stickiness
- Separates public and private traffic

## Elastic Load Balancer (ELB) Overview

- Managed load balancer
- AWS guaranteed
- AWS provides limited configuration but handles upgrades, maintenance and high availability
- Not the cheapest option (setting up your own LB can be cheaper) but easy to set up and integrated with AWS ecosystem
- Load balancers have security groups
    - From the load balancer, the appropriate security group is to allow HTTP traffic from anywhere on port 80 and HTTPS traffic on port 443
    - EC2 instances should expect traffic only from the load balancer, so the source should be the security group on the load balancer
    - A correctly working security group should show the application when pinging the LB endpoint but timeout on the EC2 endpoint
- LBs can scale but not instantaneously
- Error troubleshooting
    - 4xx errors are client induced errors
    - 5xx errors are application induced errors
    - 503 error means at capacity or no registered targets
    - If LB can't connect to instances it is a security group misconfiguration
- Monitoring
    - ELB Access logs log all access requests
    - CloudWatch Metrics give aggregate statistics

## ELB Health Checks

- Allow the ELB to know whether instances are available to handle traffic and reply to requests
- Done on a specific port and a specific route
- If HTTP response is not 200, then the instance is unhealthy
- Repeatedly done every 5 seconds

## ELB Target Groups

- Can target multiple services
    - EC2 instances (ASG managed) ⇒ HTTP
    - EC2 tasks (ECS managed) ⇒ HTTP
    - Lambda functions ⇒ HTTP translated to JSON event
    - IP Addresses ⇒ Must be private
- Health checks are at group level

## ELB Types

- All ELBs can be internal (private) or external (public)
- Classic (CLB)
    - Supports HTTP, HTTPS (Layer 5), TCP (Layer 4)
    - Health checks are HTTP or TCP
    - Fixed hostname: `name.region.elb.amazon.aws`
    - Steps for creation
        1. Choose name, VPC, internal/external and configure the listener
        2. Assign security group
        3. Configure health checks
            - Ping port and path
            - Response timeout
            - Intervals
            - Healthy/Unhealthy thresholds
- Application (ALB)
    - Support HTTP, HTTPS, WebSocket
    - Level 7 only ⇒ rule-based URI targeting
    - Load balance to multiple HTTP applications across machines
    - Load balance to multiple applications on the same machine
    - Works on target groups and can target multiple
    - Supports Redirect
    - Allows routing tables to separate target groups
        - URL-based routing
        - Hostname-based routing
        - Query String/Header-based routing
    - Optimal for microservices and container-based applications
    - Dynamic port mapping in ECS
    - Fixed hostname: `name.region.elb.amazon.aws`
    - Application servers don't see client details directly because they receive traffic from private ALB IP so they have to use extra headers if they need connection detail
        - IP ⇒ inserted in `X-Forwarded-For` header
        - Port ⇒ inserted in `X-Forwarded-Port` header
        - Protocol ⇒ inserted in `X-Forwarded-Proto` header
    - Steps for creation
        1. Choose name, listeners, VPC, AZs
        2. Choose security group
        3. Create target group (type, protocol, port)
        4. Create target group (type, protocol, port)
        5. Configure health checks
        6. Register targets
        7. Configure routing rules
- Network (NLB)
    - Supports TCP, TLS, UDP
    - Level 4 only ⇒ Network based targeting
    - Works on target groups
    - Handles millions of requests per second
    - Improves latency (~100ms) and handles long-running connections
    - One static IP per AZ and supports Elastic IPs
    - Applications can see client details (network does not get cut at the LB level) so security groups for instances have to accept TCP traffic
    - Steps for creation
        1. Choose name, listeners, VPC, AZs and IP per AZ (can be autoassigned or Elastic IP)
        2. Create target group (type, protocol, port)
        3. Configure health checks
        4. Register targets
        5. Configure routing rules

## LB Stickiness

- Option to have the same client always be redirected to the same instance behind a load balancer
- Works on CLB and ALB
- Use cookies with controlled expiration date
- Can lead to load imbalances if many requests are consistently routed to the same instance
- Works on a target group level in ALB

## Cross-zone Load Balancing

- Allows each ELB to distribute traffic across all registered instances in any AZ
- On CLB, disabled by default, but it incurs in no charges if enabled
- On ALB, always on and no charges
- On NLB, disabled by default and it incurs in charges

## Connection Draining

- Different names per ELB type
    - CLB ⇒ Connection Draining
    - ALB/NLB ⇒ Deregistration Delay
- Time to complete in-transit requests while instance is de-registering or unhealthy
- Stop sending new requests to the de-registering instance\
- Delay can be between 0 (disabled) and 3600 seconds, defaults to 300 seconds
- Delay setting should depend on length of requests, low if short requests and high if long requests

## SSL/TLS Certificates

- Enables traffic between client and LB to be encrypted in transit
- SSL ⇒ Secure Socket Layer
- TLS ⇒ Transport Layer Security, currently the mainly used ones
- Public SSL certificates are issued by Certificate Authorities (GoDaddy, Symantec, etc...)
- Certificates have expiration dates and must be renewed
- The LB uses X.509 certificate (SSL server certificate) managed through AWS Certificate Manager
- Added as default certificate on the HTTP listener
- Can specify security policies to support legacy clients
- ELB support
    - CLB
        - Only one SSL certificate
        - Must use multiple CLBs to host multiple hostnames with multiple SSL certificates
    - ALB
        - Uses SNI to handle multiple SSL certificates
    - NLB
        - Uses SNI to handle multiple SSL certificates

## Server Name Indication (SNI)

- Handles multiple SSL certificate on one web server to serve multiple websites
- Client indicates in the SSL handshake which website they want to connect to and the server automatically knows which certificate to load
- Only works on ALB, NLB and CloudFront

# Auto Scaling Groups (ASG)

## ASG Overview

- Scale out/in to match an increase/decrease in traffic load
- Ensure minimum/maximum number of instances running
- Free to use, but you pay for the launched instances
- Automatic instance registration to ELB
- IAM roles attached to ASG are inherited by EC2 instances launched by it

## ASG Attributes

- Launch configuration (old)/Launch Templates (new, can specify spot fleets)
    - AMI and Instance Type
    - EC2 User Data
    - EBS volume
    - Security Group
    - SSH Key Pair
    - New launch configurations/templates must be provided to update an ASG
- Minimum/maximum size and desired capacity
    - If an instance fails or is terminated, ASG will replace it automatically
- Network/Subnet information
- ELB information
    - Can terminate unhealthy instances and replace them
- Scaling policies
    - Scale in response to Auto Scaling Alarms
    - ASG can scale based on CloudWatch Alarms (metrics-based)
    - Metrics are across overall ASG
    - Can scale in/out
    - Can be any type of metric
        - CPU usage
        - Average Network in/out
        - ELB requests
    - Can be custom metric
        - Created via PutMetric API and sent to CloudWatch
- Scaling cooldown
    - A period between the trigger of a scaling activity and the time when a new scaling activity can take place
    - Scaling-specific cooldowns override default cooldown (300 seconds)
    - Scale-in policies benefit from shorter scaling-specific cooldowns
    - If up and down scaling is frequent, specify cooldown and CloudWatch alarm periods

## Scaling Policies Implementations

- Target Tracking Scaling
    - Set a target metric to track and scale in order to maintain that target
    - Eg: average CPU usage at 50% (scale in when below, scale out when above)]
- Simple/Step Scaling
    - Specify steps of a target CloudWatch Alarm and scale when it is triggered
- Scheduled actions
    - Specific actions at specific time frames
    - Useful for highly predictable patterns

## ASG for Solutions Architect Exam

- ASG Default Termination Policy
    1. Find AZ with most number of instances
    2. If multiple instances are available, delete the ones with the oldest launch config/template
- Lifecycle Hooks
    - As soon as an instance is launched in ASG, it is in service
    - You can perform tasks before the instance goes in service during the pending state and before the instance is terminated during the terminating state
    - Can be used either to install software or configure setting during the pending state or to retrieve logs/data before instance termination
- Launch Template vs. Launch Configuration
    - Both allow to specify AMI, instance type, key pair, security groups, tags and other attributes
    - Launch Configurations
        - Must be recreated every time
    - Launch templates can have multiple versions
        - Multiple version
        - Parameter subsets (partial configurations that can be reused or inherited)
        - Can instantiate both On-Demand and Spot instances
        - Can use T2 Unlimited Burst features
        - Is recommended by AWS

# AWS Databases

## Questions to ask when choosing a database

- Read-heavy, write-heavy or balanced?
- Throughput needs?
- Does it need to scale for short periods of times?
- How much data do you need to store and for how long?
- Is it likely to grow over time?
- What is the average size of the object stored in the database?
- How frequently are they accessed and who can access them?
- Latency requirements?
- Concurrent users?
- Data durability?
- Source of truth?
- Data model?
- Schema?
- RDBMS or NoSQL?
- Transactional, analytical or search?
- Licensing costs?

## Common Database Solutions

- RDBMS
    - RDS, Aurora
- NoSQL
    - DynamoDB, ElastiCache, Neptune
- Object Store
    - S3 or S3 Glacier
- Data Warehouse
    - Redshift, Athena
- Search
    - ElasticSearch
- Graph
    - Neptune

# Relational Database Service (RDS)

## RDS Overview

- Managed DB service for relational databases
- DBs managed by AWS in the cloud ⇒ Advantages over deploying own DB on EC2
    - Automatic OS patching
    - Automatic provisioning
    - Automatic DB software updates
    - Continuous backups with point-in-time recovery
    - Read replicas for read performance
        - Key goal is to scale reads
        - Work only with SELECT statements (read-only)
        - Read replicas can be Within AZ, Cross AZ and Cross Region
        - Async reads with eventual read consistency
        - Replicas can be transformed in fully independent DB and lose replication
        - Applications using read replicas must have connections to all read replicas
        - Main use case ⇒ Two applications using the same database for different workload
        - Network costs associated with cross-AZ/cross-Region data transfer ⇒ Cheaper to make Within AZ Read Replicas
        - Can set Read Replicas as Multi-AZ for disaster recovery
    - Multi-AZ for disaster recovery
        - Key goal is fault-tolerance
        - Sync replication ⇒ When change happens in master it happens automatically in all standby
        - Single DNS name ⇒ Automatic failover to standby ⇒ Increased availability
        - Automatic failover in AZ failure, network failure, instance or storage failure
        - Self-healing app dynamics ⇒ Automatic interventions
        - Not used for scaling, just fault-tolerance
    - Can scale vertically and horizontally but does not auto-scale
    - Backed by EBS
- Can use multiple DB engines
    - PostgreSQL
    - MySQL
    - Oracle
    - Microsoft SQL Server
    - MariaDB
    - AWS Aurora (Proprietary, fully-managed)

## RDS Backups

- Auto-enabled
- Daily full backups
- Transaction logs updated every 5 minutes
- 7 day retention, max 35 days
- DB snapshots are manually triggered by user but can be retained as long as you want

## RDS Security and Encryption

- IAM and Networking
    - Best practice is for RDS to be deployed in private subnets, not public
    - Security is driven by security groups
    - IAM policies control who can manage the RDS, using RDS API
    - IAM-based authentication for MySQL and PostgreSQL
        - Uses Auth Tokens obtained by IAM and RDS API calls
        - Auth Tokens last 15 minutes
        - Enforces network in/out SSL encryption
        - Uses IAM to manage users instead of DBMS ⇒ Easier integration with IAM Roles
- At rest encryption
    - Can encrypt master and read replicas using KMS (AES-256)
    - Defined at launch
    - Master needs to be encrypted for read replicas to be encrypted
    - Transparent Data Encryption (TDE) for Oracle and Microsoft SQL Server
- In flight encryption
    - SSL certificates
    - Provide SSL options when connecting to DB
    - If SSL needs to be enforced
        - PostgreSQL
            - `rds.enforce_ssl=1` in Parameter group of RDS Console
        - MySQL
            - `GRANT USAGE ON *.* TO 'mysqluser'@'%' REQUIRE SSL;` in the DB
- General Encryption Operations
    - RDS operates on EBS ⇒ EBS encryption rules are enforced
    - To encrypt an unencrypted RDS
        - Take snapshot of unencrypted DB
        - Encrypt the copy and restore the database from the encrypted copy
        - Migrate applications to restored version and delete the old one

## Creating an RDS database

1. Standard create (custom settings) or Easy create (recommended best practices)
2. Choose DB Engine (PostgreSQL and MySQL are the only free-tier eligible)
3. Templates
    - Production
    - Dev/Test
    - Free Tier
4. Credential Settings (regular DBMS settings such as admin, passwords and similar)
5. EC2 Instances to back the DB
6. Storage (can set automatic scaling)
7. Multi-AZ deployment (not available in free tier)
8. Connectivity (VPC , security groups, public access, AZ, ports)
9. Database options (Authentication, backups, monitoring, automatic maintenance, deletion protection)

## RDS Use Cases

- Store relational datasets (RDBMS, OLTP)
- SQL queries
- Transactional inserts/update/delete

## RDS for Solution Architect

- Operations
    - Small downtime in most operations (failover, maintenance) but some operations are manual (scaling read replicas, EC2, EBS restore)
- Security
    - KMS, security groups, IAM policies, user authorizations and SSL
- Reliability
    - Multi AZ feature, failover in case of failure
- Performance
    - Depends on EC2 instances, EBS volumes and Read Replicas types
- Cost
    - Pay per hour based on EC2/EBS

# Amazon Aurora

## Aurora Overview

- Proprietary software
- Fully managed ⇒ More expensive than RDS but more efficient
- Compatible with drivers for MySQL and PostgreSQL
- Cloud-native and optimized when compared to MySQL (5x) and PostgreSQL (3x)
- Automatically scales in 10GB increments up to 64TB ⇒ Push-button scaling
- Automatically patched with zero downtime and maintained
- Advanced monitoring features
- Backtrack ⇒ point-in-time restore without using backups
- Default multi-AZ deployment, native high availability
    - 6 copies of data across 3 AZs
    - Only needs 4/6 copies for writes, 3/6 for reads
    - Self-healing with P2P replication in case of data corruption
    - Storage is striped across 100s of volumes
- Up to 15 read replicas and super low-latency replication (sub 10ms replica lag)
    - Write to one instance and replicates to all other AZs
    - Replicas can auto-scale
    - If master fails, replicas become new master
    - Fast failover (<30s)
    - Supports Cross-Region Replication
- Aurora DB Cluster
    - Writer endpoint ⇒ DNS endpoint that always points at the master
    - Reader endpoint ⇒ DNS endpoint that helps with connection load balancing and connects automatically to all the Read Replicas
- Aurora Security
    - Very similar to RDS
    - You are responsible for correct configuration of Security Groups

## Aurora Serverless

- Automated database instantiation on a pay-per-second basis
- Scales based on usage ⇒ No capacity planning
- Good for intermittent and unpredictable workloads

## Aurora Global Database

- Best way to do Cross-Region Replication
- One primary Region for Read/Write
- Up to 5 secondary read-only Regions with less than 1 second replication lag
- Up to 16 read replica per secondary region ⇒ Decrease latency
- Promoting another region to primary has Recovery Time Objective (RTO) of < 1 min

## Aurora Use Cases

- Same as RDS, but more flexible and performant while needing less maintenance

## Aurora for Solution Architect

- Operations
    - Fully managed, so less operations
    - Auto-scaling storage
- Security
    - KMS, security groups, IAM policies, user authorizations and SSL
- Reliability
    - Multi AZ feature, highly available, Serverless options
- Performance
    - Up to 15 Read Replicas, up to 5x performance of RDS
- Cost
    - Pay per hour based on EC2/EBS

# AWS ElastiCache

## ElastiCache Overview

- Managed Redis or Memcached caches
- In-memory key-value databases with really high performance and low latency
- Reduce load on read-intensive workloads ⇒ Reads from RAM are faster
- Used for stateless applications
- Write scaling with sharding, read scaling with Read Replicas
- Multi-AZ with failover
- Automatically patched with zero downtime and maintained
- Advanced monitoring features

## Cache Security

- Does not support IAM authentication
- IAM policies on ElastiCache are for API-level security
- SSL in-flight encryption
- Redis AUTH ⇒ password/token on cluster creation
- Memcached supports SASL-based authentication

## Redis vs Memcached

- Redis ⇒ More RDS like, possible backup
    - Multi-AZ with auto-failover
    - Data durability with Redis AOF persistence
        - Every write operation received by the server is logged
        - At server startup, they are replayed
    - Read Replicas for high availability
    - Backup and restore features (Redis RDB)
- Memcached ⇒ Pure cache, built to speed up applications, lost on server down
    - Multi-node partitioning ⇒ Sharding
    - Non persistent cache
    - No backup or restore
    - Multithreaded
    - Architectural Patterns
        - Lazy Loading
            - All read data is cached (can be stale)
        - Write through
            - Update cache data when written to DB (no stale)
        - Session Store
            - Temporary session data (using time to live settings)

## ElastiCache Use Cases

- Query storage ⇒ reduce same read workloads
    - ElastiCache sits in front of an RDS
    - An application first queries ElastiCache
    - If the queried data is present in ElastiCache, it returns it
    - If the queried data is not present in Elasticache, it queries RDS and writes the query results in ElastiCache
    - Need to implement invalidation rules in the cached queries to ensure correct data is returned
- User Session ⇒ Stateless applications need to know that a user is logged in
    - An user logs in in an application
    - The application writes session data to ElastiCache
    - All other applications attempt to retrieve the session data before relogging the user in

## ElastiCache for Solution Architect

- Operations
    - Small downtime in most operations (failover, maintenance) but some operations are manual (scaling read replicas, EC2, EBS restore)
- Security
    - KMS, security groups, IAM policies, no IAM Auth but Redis Auth and SSL
- Reliability
    - Clustering, sharing, multi-AZ
- Performance
    - Sub-ms performance, read replicas for sharding, in-memory
- Cost
    - Pay per hour based on EC2/EBS

# **DynamoDB**

## DynamoDB Overview

- Fully managed serverless NoSQL data store
- Exclusively backed by SSD volumes
- Single-digit millisecond response time at any scale
- Built-in security, resiliency, fault-tolerance, durability
- Replicated across multiple AZs
- No limits to storage and throughput
- Need to provision reads and writes throughput (pay for provisioned)
- Since 2018 also on-demand capacity
- Reads are eventually consistent or strongly consistent
- Throughput can auto-scale
- Security is handled through IAM
- DynamoDB Streams integrate with Lambda

## DynamoDB Tables

- Tables, items, attribute
- No joins/relationships
- Schema-less
- Key value and documents
- Unique primary key is required
- Secondary indexes
- Query on primary key, sort key or indexes
- No table size limit
- 400KB item size limit
- Item-level TTL
- Global Table can be enabled when using DynamoDB Streams
- Can replace ElastiCache as key/value store using DAX (Acccelerated Clusters)

## DynamoDB Use Cases

- Serverless applications, distributed serverless cache
- Ad impression/clickthrough
- Gaming leaderboards
- Shopping carts
- Operational state/history (Jenkins jobs logs, CI/CD logs)
- Session/state storage

## DynamoDB for Solution Architect

- Operations
    - No operations needed, auto-scaling, serverless
- Security
    - KMS, security groups, IAM policies, and SSL in-flight
- Reliability
    - Backups, multi-AZ
- Performance
    - Single-digit ms performance at any scale
- Cost
    - Pay per provisioned capacity and storage usage, can auto-scale

# AWS Route 53

## Route 53 Overview

- Managed DNS
- Domain Name Registrar
    - Can buy domains
    - Can import 3rd party domains and use Route 53 as DNS provider by updating 3rd party registrar NS records
- AWS most common DNS records
    - A ⇒ IPv4
    - AAAA ⇒ IPv6
    - CNAME ⇒ Hostname to another hostname
    - Alias ⇒ Hostname to AWS resource
- Can use public domains (www.example.com) or private domains (resolved in your VPC)
- Different features
    - Load balancing
    - Health checks
        - Default interval is 30s (can be set to 10s for $$$)
        - 15 health checkers are launched in the backend ⇒ avg 1 request every 2 seconds
        - An endpoint is considered
            - Unhealthy if it fails 3 checks in a row
            - Healthy if it passes 3 checks in a row
        - HTTP, HTTPS and TCP checks with no SSL certificate
        - Can be integrated in CloudWatch
    - Routing policies
- Pay $0.50/month/hosted zone

## DNS TTL

- Along with the DNS response, we send a TTL that specifies how long that response will live in the browser cache
- High TTL ⇒ Less traffic on DNS but chances of outdated records
- Low TTL ⇒ More traffic on DNS but low chances for outdated records/easy to change records

## CNAME vs Alias

- CNAME
    - Point hostname to another hostname
    - Only for non-root domain (xxx.domain.com, not directly domain.com)
- Alias
    - Hostname to AWS Resource
    - Works for both root and non-root domains
    - Free, comes with health checks

## Routing Policies

- Simple
    - Use when routing a single resource
    - Can't attach health checks
    - If multiple values are returned, client chooses a ranom one
- Weighted
    - Control % of requests that go to a specific endpoint
    - Can be associated with health checks
    - Use cases
        - Test % of traffic on new application versions
        - Split traffic among regions
- Latency
    - Redirect to server with the least latency to the end user
    - Determined in terms of user to their AWS Region
    - Can be associated with health checks
- Failover
    - Redirect traffic from a primary unhealthy endpoint to a healty secondary one
    - Needs a health check associated with it
- Geolocation
    - Redirect to specific endpoints based on location
    - Different from latency because it does not check whether that endpoint has the best performance for the user
    - Best approach is to create a base case if there's no match for the user's location
    - Ideal for serving localized and location-sensitive content
- Multi-Value
    - Routing to multiple resources ⇒ More advanced version of Simple routing
    - Can attach health checks
    - Up to 8 healthy records are returned for each Multi-Value query
    - Useful in conjunction with ELB but not a substitute

# CloudFront and Global Accelerator

## CloudFront Overview

- Content delivery network
- Improves read performance by caching content in edge locations for specific TTL
- DDoS protection, integrates with AWS Shield and AWS Application Web Platform
- Can expose external HTTPS and can talk to internal HTTPS backend
- Geo Restriction
    - Can allow/deny countries from accessing your content
    - Enforced by 3rd party Geo-IP database
    - Used to enforce Copyright laws
- Great for static content that needs to be available everywhere

## CloudFront Origins

- S3 buckets
    - For distribution of files
    - Enhanced security with CloudFront Origin Access Identity (OAI) as IAM Roles
    - Can be used as a way to upload files to S3
- Custom Origin (HTTP)
    - Application Load Balancer ⇒ Security group needs to allow Edge Location IP traffic
    - EC2 Instance ⇒ Security group needs to allow Edge Location IP traffic
    - S3 Website (must enable S3 static website hosting)
    - Any HTTP backend

## CloudFront Signed URL/Cookie

- Uses policies with URL expiration, IP ranges and trusted signers to distribute paid shared content to premium users over the world
- URL should be short-lived for shared content and as long as necessary for private content
- Signed URL gives access to one file ⇒ Need one Signed URL per file
- Signed Cookie gives access to multiple files

## AWS Global Accelerator

- Uses AWS internal network to serve content from Edge Locations to global clients
- Leverages two Anycast IPs to send directly the traffic to the Edge Locations, which is then routed to your application
- Works with public or private ALB, NLB, EC2 Instances and Elastic IPs
- Consistent performance by using AWS nework, fast regional failover and intelligent routing to lowest latency
- Performs Health checks over the system
- Secure due to only 2 IPs that need to be whitelisted and AWS Shield integration for DDoS Protection
- Good for non-HTTP uses such as UDP (like gaming), IoT devices or VoIP or HTTP uses that require static IP globally

## Difference between CloudFront and Global Accelerator

- Both use Edge Locations
- Both integratw with AWS Shield for DDoS Protection
- CloudFront improves content deliver for static (cacheable) and dynamic content by serving content from the Edge Locations
- Global Accelerator proxyes requests at Edge Location to applications running in one or more Regions