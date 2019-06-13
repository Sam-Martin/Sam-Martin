+++
author = "Sam Martin"
categories = ["aws", "Terraform", "Infrastructure as Code"]
date = 0001-01-01T00:00:00Z
description = ""
draft = false
image = "/images/2016/05/2016-05-02-Terraform-io-1.png"
slug = "a-week-with-terraform"
tags = ["aws", "Terraform", "Infrastructure as Code"]
title = "A Week with Terraform"

+++

As I mentioned in a [previous post](/aws-config-intro-with-cloudformation/), AWS Config is an excellent tool for ensuring compliance across your AWS accounts, but can be challenging to set up consistently across large numbers of regions and accounts. To ease this pain I attempted to set up AWS Config [using CloudFormation](](/aws-config-intro-with-cloudformation/)) but found this challenging when using custom rules due to the necessity of uploading the Lambda functions to an S3 bucket in each region first.  
To get round this I decided to use HashiCorp's [Terraform](http://terraform.io)!

# What is Terraform?
![](/images/2016/05/2016-05-02-Terraform-io.png)
Terraform is a cloud agnostic infrastructure-as-code tool which allows you to declare the configuration of your cloud environment as a text file, version that, and manage updates by making changes to the code. It's similar to CloudFormation in a lot of ways, but allows you to control not just AWS but Azure, Google Cloud, even vSphere! 

For example, to configure an AWS resource you would write something like: 
```
resource "aws_instance" "web" {
    ami = "ami-408c7f28"
    instance_type = "t1.micro"
    tags {
        Name = "HelloWorld"
    }
}
```


# Terraform Frustrations
I want to love Terraform, I want to love it so *badly* but the 3 days I spent creating the module made me want to throw it across the room and lament what felt like a missed opportunity of a tool. 

HashiCorp are known not just for creating great tools, but incredibly cleverly designed solutions to problems that people have been trying to solve badly for a very long time. With this pedigree in mind, I'm sure that the design decisions I'm about to complain about were extremely purposeful and had exceptionally good reasons behind them. This doesn't make them any less frustrating however.

## Conditional Logic
Terraform doesn't support it at all. Sure you can do some clever stuff with [interpolation](https://www.terraform.io/docs/configuration/interpolation.html), but when you're coming to Terraform having just coded a Chef cookbook, being totally unable to embed conditional logic into your templates is maddening.
This means that - for example - you often can't have optional parameters for your modules as you don't have the flexibility in things like the [template resource](https://www.terraform.io/docs/providers/template/) to exclude chunks of your template if a variable is empty.
The way I had to get around it for things like the `custom_rule_input_parameters` parameter in the module was to force the module user to include the section of the template I'd otherwise have conditionally included/exclude. (In this case a pair of empty JSON brackets `{}` if no parameters are being passed.)

## Iteration
Terraform has a [count](https://www.terraform.io/docs/configuration/interpolation.html#using-templates-with-count) function which goes some way to allow you to create multiples of the same resource (e.g. multiple EC2 instances, lambda functions, etc.) using different variables. 
This has some major issues however: 

1. Modules don't allow you to pass arrays or hashmaps as params so you have to comma/semicolon/tab separate your values and referencing them is pretty ugly.
2. The index of the count ends up being tied to the resource, so if you add or remove a value at the beginning of your comma separated list, all the subsequent resources will change index and effectively be 'new' thereby triggering complete destruction and recreation of those existing resources.
3. You have to set the number of items of whatever you're iterating over manually. If you forget to update this you will either not create all your resources or accidentally create identical resources (depending on whether you're adding or removing respectively).
## Lack of Freeform Scripting 
This is the biggest thing coming from a Chef cookbook.
Say you want to deploy an Windows 2012 R2 Base instance from the latest Amazon AMI. In Cloudformation you have to have a pre-populated hashtable of IDs for each region which rapidly gets out of date and has to be refreshed manually. In *Terraform* on the other hand... it's... well... it's exactly the same. I was expecting to be able to throw in a reference directly to the AWS SDK or code in a nominated scripting language like Python or even call out in the OS and execute some PowerShell or Python or the AWS cmdline myself, but nope. That's not an option.  
The closest you have are, you guessed it, the interpolation functions again. These are limited to being used inside strings/variables and as mentioned above get ugly *fast*. They're also relatively limited because (presumably) HashiCorp is having to code them themselves rather than leveraging an existing scripting language's functions.

## Failed Application of  Configuration
Every time I ran `terraform apply` it felt like there was a 50/50 chance I'd get an error deploying it. This was always due to my own screw up (badly named resource, didn't supply a necessary parameter, etc.), but when that happened it often left the Terraform `.tfstate` in a state where either it didn't know about the resource it had just created, or thought it had created a resource, but hadn't.  
When this happened, the way to unstick it seemed to be to either dive into a few hundred lines of json in the `.tfstate` file and remove the reference to the failed resource, or delete the resource manually (depending on the failure type).  

In some ways this is an improvement over CloudFormation, because when that happens with CloudFormation you just have to wait for the stack to fail, delete the *whole* stack including all the resources, then recreate it and respecify all the parameters. Whereas with Terraform you can actually see which resources it thinks it has and amend the `.tfstate` as necessary. 

That said however, this resultant disconnect between the actual state of the infrastructure and the state as Terraform understands it is something I was desperately hoping Terraform would *solve* coming from CloudFormation, rather than just give me an override for. 

This flaw in a lack of validation and parity with reality significantly detracts from one of the USPs of Terraform that I was most excited about. The `plan` mode; which is supposed to give you confidence that you know what's going to happen when you hit `apply`. (This feature isn't even a USP any more as [CloudFormation now has the same functionality](https://aws.amazon.com/about-aws/whats-new/2016/03/aws-cloudformation-adds-change-sets-for-insight-into-stack-updates/).)

# Terraform Conclusion

For me these issues sink Terraform for about 80% of the use cases I had in mind for it. 

I saw two major advantages:

1. Allow engineers unfamiliar with the stack to deploy easily from a Terraform template (not really possible due to the Failed Application & Arbitrary Scripting problems).
2. Fast and flexible deployment of varying sizes of complex infrastructures (completely hobbled by the lack of Conditional Logic and Iteration)

I want something that will allow me to repeatable generate the same basic infrastructure template with different sizes/options so that I can use the same Terraform modules in multiple regions/accounts and just tweak a few parameters to suit the specific requirements.

And while this is technically possible, it's difficult, and it feels like you're working against the original vision for the tool (which I suspect you are). Terraform feels like a tool which was intended to map 1:1 resource definition to resource created. It feels like the tool that builds the infrastructure that supports flexible  and scalable solutions (e.g. AutoScaling Groups), rather than building the flexible and scalable solutions itself (e.g. EC2 instances).

# Excuses
These points and opinions were formed from working with Terraform for about a week. I am not an expert in Terraform by any stretch of the imagination and although I've attended a couple of HashiCorp User Groups I'm not really familiar enough with the community to be sure that there's not something I've missed that addresses these issues or renders them null and void.  

If some-one comments on this and tells me that there's a release coming that will fix all these problems, or that there's already another way of approaching them, I will be **extremely** happy! So... please?

