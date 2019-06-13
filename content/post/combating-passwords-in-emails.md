+++
author = "toukakoukan"
categories = ["API Gateway", "aws", "Ephemera", "lambda", "security"]
date = 2016-02-06T00:00:00Z
description = ""
draft = false
image = "https://sammart.in/wp-content/uploads/2016/02/2016-02-06-Ephemera-Demo.gif"
slug = "combating-passwords-in-emails"
tags = ["API Gateway", "aws", "Ephemera", "lambda", "security"]
title = "Combating Passwords in Emails"

+++

Security is a thorny subject. One I stay away from making sweeping statements or recommendations about whenever possible. However, I’m willing to venture one assertion.

> **Plaintext passwords in emails are a BAD IDEA**

Especially when they are the primary method of communicating an initial password to a user.  
 Who would do that though? All sensible user management system allow the user to set the password themselves, or at worst send out a one-time password reset link. Right?  
 Yes, that’s the way it *should* work. That’s not to say it always does. Sometimes your system doesn’t have user self-registration (e.g. internal AD systems), doesn’t have self-service password reset, or hell in some cases I’ve seen setups that don’t allow user password management for anyone except administrators.

The objective in these scenarios should be to retrofit a proper user secret management process to your system, or swap to one that provides that functionality as soon as possible. But sometimes that’s either not possible, or will take months/years to realise, as business cases get approved, projects get resourced, and unforeseen complications delay projects.

So what do you do in the meantime? How do you provide at least a modicum of security in your existing processes without additional cost or long setup time?


# Ephemera – One Time Password Transfer

> Ephemera: things that exist or are used or enjoyed for only a short time.

![2016-02-06 - Ephemera Demo](https://sammart.in/wp-content/uploads/2016/02/2016-02-06-Ephemera-Demo.gif)

Enter [Ephemera](https://github.com/Sam-Martin/Ephemera) a transitional AWS based one time password transfer solution I’ve open sourced to allow people to safely share passwords until they have a better way of managing secrets.(http://ephemera.sammart.in) You can find a demo setup at [ephemera.sammart.in](http://ephemera.sammart.in). (Just don’t be daft and use it for transferring real secrets!)

## What does it do?

Ephemera is an extremely simple service you can host yourself that allows you to upload text and image secrets and in return receive a URL that will work only once.

This URL can then be sent to the user receiving the password, safe in the knowledge that anyone trying to access the link after the intended user has retrieved their password will be unable to. Further, if anyone intercepts the message and retrieves the secret prior to the intended recipient, the alarm will be raised when the user attempts to retrieve the secret and finds it missing.

**WARNING:** This is *not* a ‘good’ way to manage secrets. It’s not a long term solution. I *strongly* encourage anyone thinking of using Ephemera to improve the secret management of the use-case they have by changing to a more modern product, or by retrofitting a self-service secret management system in some way.

However, Ephemera *is* better than sending passwords in plain text by email or IM, and is a very quick stop-gap solution to replace that workflow.


## How does it work?

- [S3 ](https://aws.amazon.com/s3/)(for the static portion of the site and secret storage)
- [API Gateway](https://aws.amazon.com/api-gateway/) (as the endpoint for inserting/retrieving secrets)
- [Lambda ](https://aws.amazon.com/lambda)(as the node.js logic behind API gateway)
- [Key Management Service](https://aws.amazon.com/kms/) (for the encryption of the AWS Secret Key required to sign the S3 upload URL)

You’ll note there’s no EC2 in there, which means that Ephemera is *stupidly* cheap to run.

Setup is pretty simple as well, as the [GitHub repo](https://github.com/Sam-Martin/Ephemera) comes with [instructions ](https://github.com/Sam-Martin/Ephemera/wiki/Setup-With-Terraform)and a [PowerShell script](https://github.com/Sam-Martin/Ephemera/blob/master/Install-Ephemera.ps1) that orchestrates the creation of the S3 website, API Gateway, and Lambda functions using:

- [Terraform](https://www.terraform.io/)
- [AWS API Gateway Importer](https://github.com/awslabs/aws-apigateway-importer)
- [AWS PowerShell Tools](https://aws.amazon.com/powershell/)

The top two are really interesting tools and warrant blog posts of their own, but for now suffice it to say that the only thing you have to do is install the tools, setup a KMS Key, and run the script!


## Post-Setup Improvements

There are a number of improvements that you as a user of Ephemera can make post-install, and they include:

1. Putting a [CloudFront ](https://aws.amazon.com/cloudfront/)distribution in front of the S3 bucket and using [Certificate Manager](https://aws.amazon.com/certificate-manager/) to add an SSL cert to your bucket. (All secret-transfer transactions are made using the HTTPS API Gateway endpoint, but understandably the more security-conscious users of your Ephemera install may get nervous that the static content is served unencrypted.)
2. Locking down the [index.html](https://github.com/Sam-Martin/Ephemera/blob/master/frontend/index.html) page to only be accessible from certain IPs, and setting the error page for the bucket as  [getSecret.html](https://github.com/Sam-Martin/Ephemera/blob/master/frontend/getSecret.html). This will prevent anyone not accessing the installation of Ephemera from your offices from *uploading* secrets, but still allow them to download them.
3. Add a 24hr [deletion policy](http://docs.aws.amazon.com/AmazonS3/latest/dev/object-lifecycle-mgmt.html) to your bucket, this will prevent secrets from sitting around for too long and potentially unintended persons from accessing the URL.


## Potential Future Ephemera Improvements

Although Ephemera ensures encryption in transit by leveraging the HTTPS endpoints of AWS API Gateway, it does not enforce encryption at rest. Ideally I want to use the Ephemera KMS key to encrypt the objects stored in S3. This has the advantage above-and-beyond just ticking “Encrypt S3 bucket” that even S3 admins won’t be able to access the secrets.

I also want to provide the setup script in a Linux friendly language like Python. Apart from anything this will allow me to rebuild a beta-example of Ephemera at build time using [Travis CI](https://travis-ci.org/), which would be pretty cool!

## Feedback

I’m really interested in feedback for Ephemera, especially any glaring security holes people may spot. Even if you just want help/advice setting it up, let me know, I’m keen to hear from anybody who may end up using it.
## Alternatives

Here I’m going to try and list post-hoc password self-service solutions that you can use as a longer term strategy to improving password management.

### LDAP/Active Directory

#### PWM

> PWM is an open source password self service application for LDAP directories. PWM is an ideal candidate for organizations that wish to 'roll their own' password self service solution, but do not wish to start from scratch. [Overview/Screenshots](https://docs.google.com/presentation/d/1LxDXV_iiToJXAzzT9mc1xXO0atVObmRpCame6qXOyxM/pub?slide=id.p8)

Official project page is at [https://github.com/pwm-project/pwm/](https://github.com/pwm-project/pwm/).

