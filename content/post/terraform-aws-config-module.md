+++
author = "Sam Martin"
date = 0001-01-01T00:00:00Z
description = ""
draft = true
slug = "terraform-aws-config-module"
title = "Terraform AWS Config Module"

+++

# Preamble
I've introduced Terraform and some of its disadvantages on the blog [previously](/a-week-with-terraform).
I've also talked about AWS Config and how you can use CloudFormation to configure it (and some of the challenges in that) on the blog [before as well](/aws-config-intro-with-cloudformation/).

So why go to the bother of using Terraform for this if we can do it with CloudFormation? Especially considering how disappointed I was with Terraform in my last post? 
One thing: **Lambda**. Terraform is able to create and manage Lambda functions from your disk, without having to manually usher them into a S3 bucket first, meaning you can have a small script to download and package your Lambda source files locally, then do the rest with Terraform.

#Objective
To meet my use case, my Terraform config had to do the following:  

1. Configure AWS Config's `DeliveryChannel` and `ConfigurationRecorder` resources  .
2. Create the Lambda functions for the custom rules.
3. Create the custom rules.
4. Provide the flexibility to add new custom rules easily.

# Challenge
I had one major stumbling block however, at the time of writing Terraform doesn't have a resource for AWS Config!

This means that where for other resources you can simply declare them like our ec2 example above, if you write `resource "aws_config"`, Terraform won't know what you're talking about. 
Now, ideally what we'd do to remedy this is write the resource ourselves. Terraform is open source and HashiCorp are basically the paragon of open source collaboration, so writing the resource and getting it merged should be trivial. However, I was on a very tight deadline, and learning Go, Terraform resource coding, and the appropriate testing framework just wasn't going to happen.

# Solution: Terraform Module + CloudFormation
What is much *easier* to do however is to create a [Terraform Module](https://www.terraform.io/docs/modules/usage.html).
A module in Terraform is totally identical to the standard infrastructure definition language. It's merely a way of referencing one definition from another. On its own, the module doesn't solve our problem (that there's no way of directly managing AWS Config using the Terraform AWS provider), but it does allow us to more elegantly solve this problem indirectly. 

While AWS Config isn't supported by Terraform, AWS *CloudFormation* is. Using the CloudFormation template we played with in the [previous post](/aws-config-intro-with-cloudformation/) and Terraform's [aws\_cloudformation\_stack](https://www.terraform.io/docs/providers/aws/r/cloudformation_stack.html) resource we can manage AWS Config and our custom rules, and then use Terraform resources for everything else (IAM roles, lambda functions, etc.)


So that's what I did!

![aws-config-custom-rules-terraform](/images/2016/05/2016-05-02-aws-config-custom-rules-terraform-1.png)
#### [Sam-Martin/terraform-aws-config-module](https://github.com/Sam-Martin/terraform-aws-config-module)

# Using the Module

## Initial Setup

1. Create an S3 bucket in which to store AWS Config snapshots
1. Install [Terraform](http://terraform.io)
2. Setup an AWS access/secret key in [~\\.aws\credentials](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html) with an IAM user that has permissions against:
  1. AWS Config
  2. Lambda
  3. IAM Roles
  4. CloudFormation
  5. possibly-other-things-I've-forgotten-oh-hell-just-give-it-admin

## Module configuration
For all my complaints about Terraform, I do like how easy it is to use modules (for the most part). The start of our new Terraform configuration will be a file called `main.tf` in an empty folder that contains the following:

```
variable "region" {
  type = "string"
  default = "eu-west-1"
}

provider "aws" {
  region = "${var.region}"
}

module "aws_config_rules" {
  source = "github.com/Sam-Martin/terraform-aws-config-module/module"

  region = "${var.region}"
  delivery_channel_s3_bucket_prefix = "myfirstlogs"
  delivery_channel_s3_bucket_name = "aws-config-cfn-template-example"
}

```

This is sufficient to specify which module we want to call, tell it where we want to setup AWS Config, and run it with the default rules.

## Rule Packaging
Included in the module is a little [PowerShell script](https://github.com/Sam-Martin/terraform-aws-config-module/blob/master/package-rule-lambda-functions.ps1) to download and package the custom rules we'll need. Download it to your Terraform configuration directory and run it. (After you've looked at what it does of course!)
This creates a folder called `temp` in the CWD, downloads the source for the rules listed in the script into that folder, then zips them up to equivalently named zip files.
The module looks for these files in `./temp` by default so we don't need to tell it where they are.
# Module Execution

We can now execute (with that folder as the CWD)
```
terraform get
terraform plan
```
The first command downloads the module from GitHub, the second shows us the execution plan.
![](/images/2016/05/2015-05-02---Terraform-Plan.png)

Have a peruse over the detailed output, you can safely ignore any errors about `Error parsing JSON`, it seems there are a few bugs with using CloudFormation and Template resources together, it will apply successfully anyway.

Make sure you're happy with it, then run
```
terraform apply
```
A lot of text will scroll past and then...
![](/images/2016/05/2015-05-02---Terraform-Apply.png)

You should now be able to navigate to AWS Config in the console, select the correct region and...
![](/images/2016/05/2015-05-02---AWS-Config.png)

There we are! Three rules setup for you automatically.
The defaults just scratch the surface of what AWS Config is capable of. To start creating your own rules check out the [AWS Documentation](http://docs.aws.amazon.com/config/latest/developerguide/evaluate-config_develop-rules.html), the official [GitHub repo](https://github.com/awslabs/aws-config-rules) of custom rules, and make sure you share the ones you create!

Happy automation!

