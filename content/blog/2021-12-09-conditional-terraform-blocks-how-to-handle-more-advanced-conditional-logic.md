---
title: Conditional Terraform blocks - how to handle more advanced conditional logic
date: 2021-12-08T23:52:28.476Z
description: What if you want just part of your resource to be conditional?
---
We already know the current workaround for having conditional resources in Terraform, we use the `count` attribute.

```hcl
resource "null_resource" "foo" {
   count = var.is_enabled ? 1 : 0
}
```

There's sadly no other way of doing this with Terraform at the moment, but there are some instances where you'll need the resource but not part of it.

Let's say you have an IAM assume role policy that you need to create only if you pass a certain principal (maybe you don't always create the resource the principal points to, the role creation will fail)

This is a very real usage of conditional blocks, you have your base policy, and you want to grant access to that role only if you pass a certain principal.

Let's see our base document.

```hcl
data "aws_iam_policy_document" "base_policy" {
  statement {
    actions = ["sts:AssumeRole"]

    principals {
      type        = "Service"
      identifiers = ["ec2.amazonaws.com"]
    }
  }
}
```

If the variable contains a value, then we want to create a new block in our policy that allows that account to assume this role.

We can do something like this:

```hcl
variable "role_arn" {
  default     = null
  description = "This role is allowed to assume the base role for this account."
  type        = string
}

data "aws_iam_policy_document" "base_policy" {
  statement {
    actions = ["sts:AssumeRole"]

    principals {
      type        = "Service"
      identifiers = ["ec2.amazonaws.com"]
    }
  }

  dynamic "statement" {
    for_each = var.role_arn ? [1] : []

    content {
      actions = ["sts:AssumeRole"]

      principals {
        type        = "AWS"
        identifiers = [var.role_arn]
      }
    }
  }
}
```

And that's it, there's no need for any fancy logic, in the future I hope that Terraform handles conditional operations better but meanwhile this should be enough for most use cases.