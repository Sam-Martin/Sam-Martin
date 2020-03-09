---
title: "Not Authorized to Perform Iam:PassRole"
summary: "I was troubleshooting... as my scenario was a little complicated.  
I was running Terraform in a Lambda function (as you do) and that lambda's execution role had just been given permission to assume the OrganizationAccountAccessRole as a troubleshooting step to rule out permissions issues...
"
image: "/images/2020/03/2020-03-09-cloudwatch-events-permissions.png"
date: 2020-03-09T20:30:52Z
draft: false
---

# Access Denied Exception - not authorized to perform: iam:PassRole

```
Error: Creating Cloudwatch Event Target failed: AccessDeniedException: User: 
    arn:aws:sts:::111111111111:assumed-role/OrganizationAccountAccessRole/session-name is not authorized to perform:
    iam:PassRole on resource: arn:aws:iam::111111111111:role/my_new_role
    status code: 400, request id: 11111-111-11111-111-11111

  on .terraform/modules/blah.blah.blah/cloudwatch.tf line 7, in resource "aws_cloudwatch_event_target"
```

WHAT? But... OrganizationAccountAccessRole is literally god mode...

I was troubleshooting... as my scenario was a little complicated.  
I was running Terraform in a Lambda function (as you do) and that lambda's execution role had just been given permission to assume the OrganizationAccountAccessRole as a troubleshooting step to rule out permissions issues, even though the role it had previously had `iam:PassRole` anyway.  

Terraform was doing the assuming using [AWS Provider `assume_role` config](https://www.terraform.io/docs/providers/aws/index.html#assume-role), which I thought might be complicating matters, BUT running the same terraform worked from my laptop, so it obviously wasn't the terraform config.  
That also meant it wasn't an SCP on the account, as that would have applied to my user too (and it didn't say `Explicit Deny` as SCPs tend to).  
It also wasn't a Permission Boundary, as none were configured for any of the roles involved.
It *also* wasn't the lambda role config, as giving that god-mode didn't help matters.

After much head-scratching and fruitless cloudtrail searching I realised that my Lambda function was VPC connected lambda function. It was communicating with the CloudWatch Events API using a VPC Endpoint.

That VPC endpoint had the following policy:


```
{
    "Statement": {
        "Action": ["events:*"],
        ...
    }
}
```

I had a hunch. I added `, "iam:PassRole"` to the `"Action"` array.
```
{
    "Statement": {
        "Action": ["events:*", "iam:Passrole"],
        ...
    }
}
```

Didn't work...

Wait 5 minutes, try again.

```
Apply Complete!
```

Holee shit. As if IAM permissions weren't hard enough!  
Turned out that the `iam:PassRole` call was going through the Events Endpoint, and the Events Endpoint was denying it due to the person who configured it (quite reasonably) assuming that the freaking Events Endpoint would only ever deal with `events:*` actions!

I really wish there was something in the error that showed that the permission denial was coming from an endpoint policy as that was confusing AF to figure out!
