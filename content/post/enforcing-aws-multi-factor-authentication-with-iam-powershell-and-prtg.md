+++
author = "toukakoukan"
date = 2014-09-07T00:00:00Z
description = ""
draft = false
image = "/images/2014/09/loginwithmfa.png"
slug = "enforcing-aws-multi-factor-authentication-with-iam-powershell-and-prtg"
title = "Enforcing AWS Multi-Factor Authentication with IAM, PowerShell and PRTG"

+++

# Introduction: MFA

Multi-Factor Authentication as utilised by AWS uses a TOTP (Time based One Time Password) setup with either a hardware or ‘virtual’ MFA device. The virtual device being the most commonly used, allowing you to use applications like Google Auth on your smartphone to generate passwords that are only viable for 60 seconds.

This means that if you have MFA enabled, even if someone has your password, so long as they don’t *also* have access to your (hardware or virtual) MFA device, they’re unable to login to your account.


# Introduction: AWS MFA

MFA as utilised by AWS is pretty straightforward to setup, scan a QR code, type in a couple of PINs, job done. So long as you have the right permissions.

In order to allow your IAMs users to even *setup *their MFA device you need to set a policy against their user (preferably indirectly using a group). Something like this:

```
{
	"Version": "2012-10-17",
	"Statement": [{
		"Sid": "AllowUsersToCreateDeleteTheirOwnVirtualMFADevices",
		"Effect": "Allow",
		"Action": ["iam:*VirtualMFADevice"],
		"Resource": ["arn:aws:iam::123456789012:mfa/${aws:username}"]
	}, {
		"Sid": "AllowUsersToEnableSyncDisableTheirOwnMFADevices",
		"Effect": "Allow",
		"Action": [
			"iam:DeactivateMFADevice",
			"iam:EnableMFADevice",
			"iam:ListMFADevices",
			"iam:ResyncMFADevice"
		],
		"Resource": ["arn:aws:iam::123456789012:user/${aws:username}"]
	}, {
		"Sid": "AllowUsersToListVirtualMFADevices",
		"Effect": "Allow",
		"Action": ["iam:ListVirtualMFADevices"],
		"Resource": ["arn:aws:iam::123456789012:mfa/*"]
	}, {
		"Sid": "AllowUsersToListUsersInConsole",
		"Effect": "Allow",
		"Action": ["iam:ListUsers"],
		"Resource": ["arn:aws:iam::123456789012:user/*"]
	}]
}
```
Where *123456789012 *is your AWS account ID.

Okay, so far so good. Your AWS users can set their own MFA devices. But currently whatever other privileges you’ve given them are usable even if they *haven’t* setup an MFA device for their account, meaning their account is a security vulnerability. Best put pay to that!

```
{
	"Version": "2012-10-17",
	"Statement": [{
		"Sid": "AllowUsersToCreateDeleteTheirOwnVirtualMFADevices",
		"Effect": "Allow",
		"Action": ["iam:*VirtualMFADevice"],
		"Resource": ["arn:aws:iam::123456789012:mfa/${aws:username}"]
	}, {
		"Sid": "AllowUsersToEnableSyncDisableTheirOwnMFADevices",
		"Effect": "Allow",
		"Action": [
			"iam:DeactivateMFADevice",
			"iam:EnableMFADevice",
			"iam:ListMFADevices",
			"iam:ResyncMFADevice"
		],
		"Resource": ["arn:aws:iam::123456789012:user/${aws:username}"]
	}, {
		"Sid": "AllowUsersToListVirtualMFADevices",
		"Effect": "Allow",
		"Action": ["iam:ListVirtualMFADevices"],
		"Resource": ["arn:aws:iam::123456789012:mfa/*"]
	}, {
		"Sid": "AllowUsersToListUsersInConsole",
		"Effect": "Allow",
		"Action": ["iam:ListUsers"],
		"Resource": ["arn:aws:iam::123456789012:user/*"]
	}]
}
```

Now we’re giving the user full access to everything but *only* if they have authenticated with MFA. So if they login with just a password and try to access, e.g. EC2, they’ll get a big fat access denied.

[![accessdeniedwithoutmfa](/images/2014/09/accessdeniedwithoutmfa1.png)](/images/2014/09/accessdeniedwithoutmfa1.png)

Great! So they go and setup their MFA device, logout, login again with MFA.

[![loginwithmfa](/images/2014/09/loginwithmfa.png)](/images/2014/09/loginwithmfa.png)

And voila! Access allowed.

[![accessallowedwithmfa](/images/2014/09/accessallowedwithmfa1.png)](/images/2014/09/accessallowedwithmfa1.png)

Which is great! Really secure, can’t get in with that policy without using using MFA.

But what if someone sets up another policy (which itself is lovely and granular, preserves the principle of least privilege) but forgets the MFA constraint? When you get into more numerous and complicated policies attached variously to groups, users, etc. it becomes cumbersome to audit them all for compliance even with automation.

Further, what happens when someone gets woken up on call, forgets all about MFA for this particular AWS account (which may well be one of a dozen or so they’re involved with) then gets access denied when he tries to login. Will he know to setup MFA? Or will he wake up someone to give him “the right access” to the system?

In any case, until AWS allows MFA to be part of the ‘password policy’ and prompts you to set it up as soon as you login for the first time (and even potentially afterwards depending on how paranoid you are), there’s a need to ensure all your users have MFA setup from the get-go.


# The Monitoring

I have the pleasure of using [PRTG for monitoring](http://www.paessler.com/prtg). A capable little tool, but the following code can be adapted for any tool running on Windows.

```
[CmdletBinding()]
Param(
    [parameter(Mandatory=$true)]
    [string]$accessKey,
    [parameter(Mandatory=$true)]
    [string]$secretKey
)

# Grab the current working directory of the script for the purposes of loading the DLL
$scriptWorkingDirectory = Split-Path -Path $MyInvocation.MyCommand.Definition -Parent

# Ensure you use the .NET 4.5 DLL not the .NET 3.5 DLL from the AWS .NET SDK
# Load AWS API DLL
$AWSAPIFiles = @(
    "$scriptWorkingDirectoryAWSSDK.dll"
)
foreach($apiFile in $AWSAPIFiles){

    # Try loading the DLL
    Write-Verbose "Loading $apiFile";
    try{
        $fileStream = ([System.IO.FileInfo] (Get-Item $apiFile)).OpenRead();
    }catch{
        Write-Error $_.exception.message;
        Exit 1;
    }

    # Read the contents of the DLL
    $assemblyBytes = New-Object byte[] $fileStream.Length
    $fileStream.Read($assemblyBytes, 0, $fileStream.Length) | out-null;
    $var= $fileStream.Close()

    # Load the library
    [System.Reflection.Assembly]::Load($assemblyBytes) | out-null;
}

# Set the AWS Access Key and Secret Key for authentication using the .NET SDK
[System.Configuration.ConfigurationManager]::AppSettings["AWSAccessKey"] = $accessKey
[System.Configuration.ConfigurationManager]::AppSettings["AWSSecretKey"] = $secretKey

# Connect to the AWS API
Write-Verbose "Connecting to AWS API";
$client= New-Object -TypeName Amazon.IdentityManagement.AmazonIdentityManagementServiceClient;

# Fetch the list of users that have passwords but not MFA
Write-Verbose "Fetch users that have passwords, but no MFA";
$mfadevices = @()
$usersWithoutMFA = $client.listUsers().ListUsersResult.Users | ?{

    # Ensure the user has a password (if they only have a secret key, they don't need MFA)
    try{
        $client.GetLoginProfile($_.username) | Out-Null;
    }catch{
        return $false;
    }

    # Return false if they don't have MFA (otherwise we don't care about them as they're doing the right thing!)
    return !$client.ListMFADevices($_.username).MFADevices;
}

# Output to PRTG
Write-Verbose "Output in a PRTG friendly format (XML)";
Write-Host "
<prtg>
    <result>
        <channel>Number of users without MFA devices registered</channel>
        <value>$(($usersWithoutMFA | Measure-Object).count)</value>
    </result>
    <Text>$(($usersWithoutMFA | select -expandProperty "Username") -join "; ")</Text>
</prtg>";


```

 
In order to execute this you need the following pre-requisites:

1. The .NET 4.5 AWSSDK.dll from the [AWS .NET developer’s SDK](http://aws.amazon.com/sdk-for-net/) must be housed in the same directory as the .ps1
2. PowerShell 4.0 or higher must be installed on the PRTG Probe
3. .NET 4.5 must be installed on the PRTG probe executing the custom sensor
4. A user with at least the following privileges in AWS:

```
{
	"Version": "2012-10-17",
	"Statement": [{
		"Sid": "Stmt1410864868000",
		"Effect": "Allow",
		"Action": [
			"iam:ListUsers",
			"iam:ListMFADevices",
			"iam:GetLoginProfile"
		],
		"Resource": [
			"arn:aws:iam::123456789012:*"
		]
	}]
}
```
Where, again *123456789012* is replaced with your account ID.

In order to get the .NET 4.5 AWSSDK.dll from the AWS .NET developer’s SDK just install the SDK on your machine, then copy AWSSDK.dll from `C:\Program Files (x86)\AWS SDK for .NET\bin\Net45` to the directory your script lives in.

This directory should be under your PRTG probe’s *Custom SensorsExeXML *directory.

Once you’ve done that, you can create a Script/Exe custom sensor in PRTG pointing at your new .ps1 file like so:

[![PRTGsensorMFA](/images/2014/09/prtgsensormfa.png)](/images/2014/09/prtgsensormfa.png)

Setting the arguments to reflect the access and secret keys of the AWS user you created earlier.

Once that’s done, you’ll have a sensor that shows the names of the users in your AWS account that have a password, but no MFA device. Great! But how do we alert on that? As when that devices goes to an error state, the message will be replaced with an error message!

No problem, just create a *factory sensor* that references the first sensor, then create a threshold on the channel.

*Create Sensor > Factory Sensor > Properties*
```
#<factory sensor channel ID>:<factory sensor name> Channel(<custom sensor id>,<custom sensor channel>)

#1:Users without MFA on AWS Channel(10101,2)
```
Then set the threshold against the channel like so:  
[![mfachannelthreshold](/images/2014/09/mfachannelthreshold.png)](/images/2014/09/mfachannelthreshold.png)  
 Voila! You will be alerted whenever you have a user that has a password, but no MFA device associated!

How do you handle this issue in your environment? Any suggestions on how to do this better? Please let me know in the comments!


# Further Reading

[StackOverflow – Can you require MFA for AWS IAM accounts?](http://serverfault.com/questions/483183/can-you-require-mfa-for-aws-iam-accounts)

[AWS Docs – Configuring and Managing a Virtual MFA Device for Your AWS Account (AWS Management Console)](http://docs.aws.amazon.com/IAM/latest/UserGuide/GenerateMFAConfigAccount.html)

[JeffW@AWS – Allow your user to self-manage a virtual MFA](https://forums.aws.amazon.com/thread.jspa?messageID=424459#424459)

