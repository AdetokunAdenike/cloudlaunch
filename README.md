# CloudLaunch Assignment

## Overview
**CloudLaunch** is a lightweight project showcasing a basic company website and private document storage using AWS core services. This assignment demonstrates my understanding of AWS S3, IAM, and VPC fundamentals, including secure access control and logical network design.

The assignment consists of two main tasks:

* Task 1: **Static Website Hosting and IAM Permissions**
* Task 2: **VPC Design and Security Groups**

---

## Task 1: Static Website Hosting Using S3 + IAM User

### 1. S3 Buckets Created

| Bucket Name | Purpose | Access |
|------------|---------|--------|
| `cloudlaunch-site-bucket` | Hosts the static website (HTML/CSS/JS) | Public read-only |
| `cloudlaunch-private-bucket` | Stores internal private documents | `cloudlaunch-user` GetObject/PutObject only |
| `cloudlaunch-visible-only-bucket` | Can be listed by IAM user but contents are private | No object access |

### 2. Website Setup

- Uploaded a simple **`index.html`** with CSS and JavaScript to `cloudlaunch-site-bucket`.
- Enabled **static website hosting**.
- The website is accessible via the HTTP endpoint:
  ```
  http://cloudlaunch-site-bucket-lara.s3-website-eu-west-1.amazonaws.com/
  ```
- Configured CloudFront distribution (HTTPS and global caching).  
  ```
  https://d2zlbethk8fxca.cloudfront.net/
  ```

### 3. IAM User: `cloudlaunch-user`

#### Custom Policy JSON
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ListBucketOnAllThree",
            "Effect": "Allow",
            "Action": "s3:ListBucket",
            "Resource": [
                "arn:aws:s3:::cloudlaunch-site-bucket-lara",
                "arn:aws:s3:::cloudlaunch-private-bucket-lara",
                "arn:aws:s3:::cloudlaunch-visible-only-bucket-lara"
            ]
        },
        {
            "Sid": "GetOnSiteObjects",
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::cloudlaunch-site-bucket-lara/*"
        },
        {
            "Sid": "GetPutOnPrivateObjects",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::cloudlaunch-private-bucket-lara/*"
        },
        {
            "Sid": "ExplicitDenyDeletesEverywhere",
            "Effect": "Deny",
            "Action": [
                "s3:DeleteObject",
                "s3:DeleteObjectVersion"
            ],
            "Resource": [
                "arn:aws:s3:::cloudlaunch-site-bucket-lara/*",
                "arn:aws:s3:::cloudlaunch-private-bucket-lara/*",
                "arn:aws:s3:::cloudlaunch-visible-only-bucket-lara/*"
            ]
        },
        {
            "Sid": "LetUserSeeBucketListInConsole",
            "Effect": "Allow",
            "Action": "s3:ListAllMyBuckets",
            "Resource": "*"
        }
    ]
}
```

#### Testing IAM User
- Logged in as `cloudlaunch-user` to verify:
  - Upload/download to **private bucket**:
  - Access to **visible-only bucket** objects (only bucket listing visible)  
  - Access to **site bucket** objects (read-only)
  - CloudFront and S3 bucket listing (as appropriate)

---

## Task 2: VPC Design for CloudLaunch Environment

### 1. VPC

- Name: `cloudlaunch-vpc`
- CIDR: `10.0.0.0/16`

### 2. Subnets

| Subnet Name | CIDR | Type | Purpose |
|------------|------|------|---------|
| `cloudlaunch-public-subnet` | 10.0.1.0/24 | Public | Load balancers / internet-facing services |
| `cloudlaunch-app-subnet` | 10.0.2.0/24 | Private | Application servers |
| `cloudlaunch-db-subnet` | 10.0.3.0/28 | Private | Database / RDS-like services |

---

### 3. Internet Gateway

- Created **`cloudlaunch-igw`** and attached to `cloudlaunch-vpc`.
- Public subnet route table points `0.0.0.0/0` to this IGW.

---

### 4. Route Tables

| Route Table | Associated Subnet | Routes |
|------------|-----------------|-------|
| `cloudlaunch-public-rt` | `cloudlaunch-public-subnet` | `0.0.0.0/0 → cloudlaunch-igw` |
| `cloudlaunch-app-rt` | `cloudlaunch-app-subnet` | local only (private) |
| `cloudlaunch-db-rt` | `cloudlaunch-db-subnet` | local only (private) |

---

### 5. Security Groups

| Security Group | Inbound Rules |
|----------------|---------------|
| `cloudlaunch-app-sg` | HTTP (80) from VPC `10.0.0.0/16` only |
| `cloudlaunch-db-sg` | MySQL (3306) from `cloudlaunch-app-sg` only |

---

### 6. IAM User Read-only Access to VPC

#### Used AWS Managed Policy for VPC Read-only: `AmazonVPCReadOnlyAccess`

- Allows `cloudlaunch-user` to **view/list** all VPC resources but not modify them.
---

## S3 Website Links

- Public site:  
  ```
  http://cloudlaunch-site-bucket-lara.s3-website-eu-west-1.amazonaws.com/
  ```
- CloudFront:
```
https://d2zlbethk8fxca.cloudfront.net/
```