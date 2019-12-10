+++
author = "toukakoukan"
date = 2016-04-24T00:00:00Z
description = ""
draft = false
slug = "aws-config-intro-with-cloudformation"
title = "AWS Config Intro with CloudFormation"

+++

[AWS Config](https://aws.amazon.com/config/) is a tool from Amazon designed to help you audit your AWS infrastructure for best practice and security adherence.

It’s ideal for maintaining an overview across multiple accounts (especially Dev accounts) to ensure that people aren’t accidentally leaving 3389 or 22 open to the entire world, and similar security ‘faux pas’.  
 Multiple account management for AWS is still quite a shaky experience. Managing users, billing, connectivity, etc. is a fractured and often confusing process, so anything that offers a helping hand for standardisation is extremely welcome.


## How does it work?

AWS Config will periodically push a “snapshot” (read gigantic json file) of your entire AWS configuration (instance IDs, security groups, bucket policies, IAM users, *everything*) to an S3 bucket, or SNS topic of your choice. This will then trigger Rules to evaluate the configuration and ensure that it’s in compliance. These rules can either be “Managed Rules” or “Custom Rules”. (There are only a handful of managed rules currently, and so the *real* power of AWS Config comes from the custom rules.)

The rules can also be triggered by the configuration change of an AWS resource (e.g. AWS::EC2::Instance) which will then evaluate the configuration of the specific resource that’s changed.

The snapshots allow you to look back through your resource’s “Config Timeline”, to see what configuration changed and when.

[![2016-04-24 - AWS Config Timeline](/wp-content/uploads/2016/04/2016-04-24-AWS-Config-Timeline-1024x438.png)](/wp-content/uploads/2016/04/2016-04-24-AWS-Config-Timeline.png)


## Managed Rules & Custom Rules

Okay, great! So what rules can we configure?

At the time of writing there are 7 [Managed Rules](http://docs.aws.amazon.com/config/latest/developerguide/evaluate-config_use-managed-rules.html) which cover some common compliance scenarios.

- CLOUD_TRAIL\_ENABLED
- EIP_ATTACHED
- ENCRYPTED_VOLUMES
- INCOMING\_SSH_DISABLED
- INSTANCES\_IN_VPC
- REQUIRED_TAGS
- RESTRICTED\_INCOMING_TRAFFIC

Each of these have different parameter requirements and are triggered on either Configuration Change or Periodic evaluation, but they’re easy to use and can be setup very simply by working through the wizard when you first go into the AWS Config console.

Okay, good start! But how do we go about extending these? Why, Lambda of course!  
 You can supplement the Managed Rules by creating a lambda function ([following the tutorial](http://docs.aws.amazon.com/config/latest/developerguide/evaluate-config_develop-rules_nodejs.html)), creating a Custom Rule in the AWS Config UI, and supplying the ARN of the Lambda function.  
 Better yet, there’s a whole host of Custom Rules being collaborated on in the [awslabs/aws-config-rules Github](https://github.com/awslabs/aws-config-rules/) Repository which you can use as a guide, or just straight up consume as they are.

Downside to this is of course that you can only run custom rules in regions which support Lambda, but that *has* to become universal soon… right?


## AWS Config Management with CloudFormation

So we’ve got a great framework for evaluating compliance across our AWS accounts, but if like me you want to roll a standard set of rules out over 50~ AWS accounts, you don’t want to do this clicking through the GUI, especially as AWS Config Rules are *per region*!

So you dive into the [CloudFormation documentation](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/) and discover there are two resources you need to create first.

1. [AWS::Config::DeliveryChannel](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-config-deliverychannel.html)
2. [AWS::Config::ConfigurationRecorder](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-config-configurationrecorder.html)

The first thing you note on those two pages is the following text:

> If you create this resource, you must also create or have an AWS::Config::DeliveryChannel/AWS::Config::ConfigurationRecorder resource already running in your account. These two interdependent resources must be present to successfully create both resources.

Huh, interesting, oh well, I’ll just create them both without dependencies on each other, that should be fine right? If you setup AWS Config initially with CFN, and get the stack creation right the first time *maybe*.  
 However, if you’ve configured AWS Config previously, you can’t guarantee that your CFN stack creation will succeed because the DeliveryChannel defines the destination S3 bucket for the Config Snapshots, and the ConfigurationRecorder defines the IAM role which provides access to that bucket.

This means that if the DeliveryChannel gets created first and the IAM role which is specified in the existing ConfigurationRecorder does not have access to the bucket the DeliveryChannel defines, it will fail to create!  
 Conversely if the ConfigurationRecorder gets created first and the IAM role which is specified in the new ConfigurationRecorder does not have access to the bucket the old DeliveryChannel defines, it will fail to create.  
 Therefore using just the AWS Config CloudFormation Resources it’s impossible to create a reliable stack that changes from one bucket destination to another.

So what do we do when we get cyclic dependencies with CloudFormation? We bring in Lambda and Custom Resources of course!  
 To save the work of going through the whole process, I’ve created an example here

[Sam-Martin/aws-config-cfn-template-example](https://github.com/Sam-Martin/aws-config-cfn-template-example/blob/master/AWS%20Config%20Base.template).

[![2016-04-24 - aws-config-cfn-template-example](/wp-content/uploads/2016/04/2016-04-24-aws-config-cfn-template-example-1-1024x887.png)](/wp-content/uploads/2016/04/2016-04-24-aws-config-cfn-template-example-1.png)

Basically what the above does is create and trigger a Lambda Function to delete any existing DeliveryChannels and and stops the DeliveryRecorder so the path is cleared for the AWS Config Resources in the rest of the template to configure them as desired.

I wouldn’t call it terribly elegant, but it certainly works in all scenarios I’ve tested it. If anyone has found another way to make the AWS Config CloudFormation resources work reliably in all circumstances I’d be very happy to hear from you!


## Caveat – No Custom Rules… Or are there?

The downside to the above example is that it only provides an example of how to work with Managed Rules. Custom Rules are much harder to configure with CloudFormation unless you embed each rule in the CloudFormation template itself, or are willing to populate an S3 bucket in the same region with zip files containing the rules you want.

There is a way around this however! I’ve created a Terraform Module at [Sam-Martin/terraform-aws-config-modules](https://github.com/Sam-Martin/terraform-aws-config-module) that allows you to do just that!

I’ll write a blog post just on that at a later date, but in the meantime you can probably puzzle it out for yourself from the Readme ! :)
Happy automation everyone!

