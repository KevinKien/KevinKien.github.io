---
layout: post
title: Security for KMS from Terraform Config
date: 2024-09-13
categories: ["AWS Security", "KMS"]
---

# Summary

This post is will use Terraform config from [terraform/aws/kms.tf](https://github.com/bridgecrewio/terragoat/blob/master/terraform/aws/kms.tf). I will perform building checklist security for KMS and after review config kms.tf and give out result misconfigration from config kms.tf. Finnal, i will provide solution and config prevent for Terraform config.

# Building checklist
I have a checklist with best practice include:
- Key rotation
- KMS key policy
- Deletion window
- Use SSL/TLS

# KMS Terraform Config

```
resource "aws_kms_key" "logs_key" {
  # key does not have rotation enabled
  description = "${local.resource_prefix.value}-logs bucket key"

  deletion_window_in_days = 7
  tags = {
    git_commit           = "d68d2897add9bc2203a5ed0632a5cdd8ff8cefb0"
    git_file             = "terraform/aws/kms.tf"
    git_last_modified_at = "2020-06-16 14:46:24"
    git_last_modified_by = "nimrodkor@gmail.com"
    git_modifiers        = "nimrodkor"
    git_org              = "bridgecrewio"
    git_repo             = "terragoat"
    yor_trace            = "cd8fa2a7-4868-4cd1-993d-da4644808ce5"
  }
}

resource "aws_kms_alias" "logs_key_alias" {
  name          = "alias/${local.resource_prefix.value}-logs-bucket-key"
  target_key_id = "${aws_kms_key.logs_key.key_id}"
}
```
