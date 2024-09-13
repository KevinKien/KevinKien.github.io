---
layout: post
title: Security for S3 from Terraform Config
date: 2024-09-13
categories: ["AWSSecurity"]
---

# Summary

This post is will use Terraform config from [https://github.com/bridgecrewio/terragoat/blob/master/terraform/aws/ec2.tf](https://github.com/bridgecrewio/terragoat/blob/master/terraform/aws/s3.tf). I will perform building checklist security for S3 and after review config s3.tf and give out result misconfigration from config s3.tf. Finnal, i will provide solution and config prevent for Terraform config.

# Building checklist

With S3 i have checklist for audit include: 
- Bucket is public
- Use SSL/TLS on the connection
- Encrypt data
- Access control with IAM policies and bucket policies
- Enable logging and versioning

# S3 Terraform config

I have Terraform config example: 

```
resource "aws_s3_bucket" "data" {
  # bucket is public
  # bucket is not encrypted
  # bucket does not have access logs
  # bucket does not have versioning
  bucket        = "${local.resource_prefix.value}-data"
  force_destroy = true
  tags = merge({
    Name        = "${local.resource_prefix.value}-data"
    Environment = local.resource_prefix.value
    }, {
    git_commit           = "4d57f83ca4d3a78a44fb36d1dcf0d23983fa44f5"
    git_file             = "terraform/aws/s3.tf"
    git_last_modified_at = "2022-05-18 07:08:06"
    git_last_modified_by = "nimrod@bridgecrew.io"
    git_modifiers        = "34870196+LironElbaz/nimrod/nimrodkor"
    git_org              = "bridgecrewio"
    git_repo             = "terragoat"
    yor_trace            = "0874007d-903a-4b4c-945f-c9c233e13243"
  })
}
```

# Audit config

I will perform review security for each resource. The first with bucket `data`. 

```
resource "aws_s3_bucket" "data" {
  # bucket is public
  # bucket is not encrypted
  # bucket does not have access logs
  # bucket does not have versioning
  bucket        = "${local.resource_prefix.value}-data"
  force_destroy = true
  tags = merge({
    Name        = "${local.resource_prefix.value}-data"
    Environment = local.resource_prefix.value
    }, {
    git_commit           = "4d57f83ca4d3a78a44fb36d1dcf0d23983fa44f5"
    git_file             = "terraform/aws/s3.tf"
    git_last_modified_at = "2022-05-18 07:08:06"
    git_last_modified_by = "nimrod@bridgecrew.io"
    git_modifiers        = "34870196+LironElbaz/nimrod/nimrodkor"
    git_org              = "bridgecrewio"
    git_repo             = "terragoat"
    yor_trace            = "0874007d-903a-4b4c-945f-c9c233e13243"
  })
}
```

In the config, not config access list so anyone can access to bucket. When bucket public, anyone can be access and get all data in the bucket. 
Data not config encrypt with KMS. So when if have anyone can be access to bucket on AWS, their can read all data of in the bucket. 
Config not config access log so when perform audit access to bucket, if haven't access log auditor will can't check log. 
When not config version for bucket, when file deteled is difficult can restore.

# Config for security

- Config ACL change bucket public to private
```
acl    = "private"
```

- Config access control with IAM policies and bucket policies
```
policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Effect = "Allow",
        Principal = {
          AWS = "arn:aws:iam::123456789012:role/your-role" # IAM Role or IAM User ARN
        }
        Action   = "s3:*"
        Resource = [
          "${aws_s3_bucket.restricted_bucket.arn}",
          "${aws_s3_bucket.restricted_bucket.arn}/*"
        ]
      }
    ]
  })
```

- Encrypt with KMS
```
server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm     = "aws:kms"
        kms_master_key_id = "${aws_kms_key.logs_key.arn}"
      }
    }
  }
```

- Enable logging and versining
```
versioning {
  enabled = true
}
logging {
  target_bucket = "${aws_s3_bucket.logs.id}"
  target_prefix = "log/"
}
```

- Enable SSL/TLS on the connection
```
policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Effect = "Deny",
        Principal = "*",
        Action = "s3:*",
        Resource = [
          "${aws_s3_bucket.secure_bucket.arn}",
          "${aws_s3_bucket.secure_bucket.arn}/*"
        ],
        Condition = {
          Bool = {
            "aws:SecureTransport" = "false" # deny if access to bucket with HTTP
          }
        }
      }
    ]
  })
```

# Finnal config with security
```
resource "aws_s3_bucket" "data" {
  bucket        = "${local.resource_prefix.value}-data"
  acl    = "private"
  force_destroy = true
  versioning {
    enabled = true
  }
  logging {
    target_bucket = "${aws_s3_bucket.logs.id}"
    target_prefix = "log/"
  }
  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm     = "aws:kms"
        kms_master_key_id = "${aws_kms_key.logs_key.arn}"
      }
    }
  }
  policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      # Allow Access for specific IAM Role/User
      {
        Effect = "Allow",
        Principal = {
          AWS = "arn:aws:iam::123456789012:role/your-role" # IAM Role or IAM User ARN
        }
        Action   = "s3:*" # Allow all actions, modify as per your need
        Resource = [
          "${aws_s3_bucket.secure_bucket.arn}",
          "${aws_s3_bucket.secure_bucket.arn}/*"
        ]
      },

      # Deny access if not using SSL/TLS (SecureTransport)
      {
        Effect = "Deny",
        Principal = "*", # Applies to all users
        Action   = "s3:*", # Deny all actions if accessed without SSL/TLS
        Resource = [
          "${aws_s3_bucket.secure_bucket.arn}",
          "${aws_s3_bucket.secure_bucket.arn}/*"
        ],
        Condition = {
          Bool = {
            "aws:SecureTransport" = "false" # Deny if access over HTTP
          }
        }
      }
    ]
  })
}
  tags = merge({
    Name        = "${local.resource_prefix.value}-data"
    Environment = local.resource_prefix.value
    }, {
    git_commit           = "4d57f83ca4d3a78a44fb36d1dcf0d23983fa44f5"
    git_file             = "terraform/aws/s3.tf"
    git_last_modified_at = "2022-05-18 07:08:06"
    git_last_modified_by = "nimrod@bridgecrew.io"
    git_modifiers        = "34870196+LironElbaz/nimrod/nimrodkor"
    git_org              = "bridgecrewio"
    git_repo             = "terragoat"
    yor_trace            = "0874007d-903a-4b4c-945f-c9c233e13243"
  })
}
```
