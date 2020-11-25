---
title: "Terraform - Psuedu Nested For-Loop and Usergen Release"
date: 2017-11-02T12:32:30-05:00
draft: false
tags: ["windows", "terraform", "aws" ]
---

So. I am using Terraform to generate a bunch of users and wanted to modularize it as much as possible. I wanted to get to a point where I could provide a set of `aws_iam_user` objects and `aws_iam_user_policy_attachments` and have the module cleanly handle this. 
<!--more-->
Normally, you'd do something like:

_PsuedoCode:_

```
for user in user_names {
    createUser(user.Name)
    for policy in iam_policies {
        attachPolicy(user.Name, policy.id)
    }
}
```

But we dont have for loops; just the `count` property.

So we have to get funky and figure out how to iterate over ALL possible combinations of two terraform lists for this to work.

Lets start with how we declare the users:

```
module "users" {
    source = source = "walked/usergen/aws"
    pgp_key = "keybase:walked"
    user_names = ["Test1","test2"]
    iam_policies = ["arn:aws:iam::aws:policy/IAMUserChangePassword", "${aws_iam_policy.example_policy.arn}"]    
}
```

Simple enough; I want two users (Test1, Test2) and both user to have two different IAM policies attached. This means that I technically need to create 4 policy attachments; one for each policy for each user. 

So; how do we tackle this? Let's walk through it.

First; we have to iterate over all combinations (multiple number of users by number of policies):

```
resource "aws_iam_user_policy_attachment" "policy_attach" {
	count         = "${length(var.iam_policies) * length(var.user_names)}"
```

Next; we need to make sure we end up with usable user indexes. This will take the count (in our example; we're going to have a count of 4; so `count.index` values of `0, 1, 2, 3` and we need to be sure that we're going to get the index results of `0, 0, 1, 1`):

```      
    user          = "${var.user_names[count.index % length(var.user_names)]}"
```

Finally; we need to get the possible policies for each user and index. We take advantage of integer truncation to get the desired result here:

```
	policy_arn    = "${var.iam_policies[(count.index) / length(var.user_names)]}"
```

This gets us a values of `0, 1, 0, 1` as we iterate. It'll scale with the number of users/policies. 

Full snippet and links:

```
resource "aws_iam_user_policy_attachment" "policy_attach" {
    count         = "${length(var.iam_policies) * length(var.user_names)}"
    user          = "${var.user_names[count.index % length(var.user_names)]}"
    policy_arn    = "${var.iam_policies[(count.index) / length(var.user_names)]}"
    depends_on    = ["aws_iam_user.this"]
}
```

**GitHub:** <https://github.com/walked/terraform-aws-usergen>

**Terraform Registry:** <https://registry.terraform.io/modules/walked/usergen/aws/1.0.3>


