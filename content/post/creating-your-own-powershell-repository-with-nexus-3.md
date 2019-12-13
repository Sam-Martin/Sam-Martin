+++
author = "Sam Martin"
date = 2017-04-09T00:00:00Z
description = ""
draft = false
image = "/images/2017/04/Nexus-Splash-Page-2.png"
slug = "creating-your-own-powershell-repository-with-nexus-3"
title = "Creating your own PowerShell Repository with Nexus 3"
aliases = ['/creating-your-own-powershell-repository-with-nexus-3/']
+++

# Intro
The introduction of [PowerShell Gallery](https://powershellgallery.com) in PowerShell 5.0 is something that the Windows world has been craving for a long time (alongside [Chocolatey](https://chocolatey.org), though I have somewhat mixed feelings about Chocolatey).  
The PowerShell equivilent to `pip`, `gem`, `npm`, it allows you to install community made PowerShell modules with a single command (`Install-Module` to be precise!).  
This is a great way to save yourself from spending hours re-inventing the wheel and easily consume code that someone else has slaved away to make work already. On top of that, it's a really easy way of moving modules around. The traditional way, consuming PowerShell modules directly from your source control system, can be a real problem. You may not wish to install Git where you need the module, you may not have access to the source control system from where you need the module, or you may end up pulling down hundreds of megs of irrelevant files depending on how your repositories are structured.  
However, you probably don't want to use PSGallery for anything but the most community-oriented of your modules, as they're probably solving problems specific to your environment, contain revealing information about your infrastructure, and/or reveal your slightly-less-than-best-practice code to the world (though that's a motivation to get your code up to scratch and out there really!).

# Nexus OSS
Enter [Nexus](https://www.sonatype.com/download-oss-sonatype).  
Those of you familiar with the fundamentals of PowerShell PackageManagement will know that in the background it's just handling the packing, unpacking, upload, and download of [NuGet ](https://www.nuget.org/) packages, and that there are a hundred and one tutorials on [how to setup a basic NuGet server for PowerShell](http://lmgtfy.com/?q=setup+nuget+server+for+powershell).  
Which is great! But the advantage of Nexus is that it supports a whole host of package repositories other than NuGet, and as there's a non-zero chance that I'm going to need those in the future, I'd rather setup a central package repository than a bunch of different ones!

# Installing Nexus
```
iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
choco install nexus-repository
``` 
Done. [Any questions](https://chocolatey.org/faq#why-chocolatey)?

# Configuring Nexus
Now Nexus is installed on your server, you can go to http://localhost:8081 and see the following.

![Nexus Splash Page](/images/2017/04/Nexus-Splash-Page-1.png)

1) Click on the `Sign In` button in the top right corner and use the following credentials.
**User:** admin  
**Pass:** admin123

2) Now click the little cog icon to go into settings, click Repositories, then Create Repository
![Create Repository](/images/2017/04/Nexus-Create-Repository.png)

3) Select `nuget (hosted)` from the recipies list.  
4) Name your module something memorable (e.g. powershell-modules).  
5) Go to `Realms` in the settings menu (visible in the above screenshot).
6) Make the `NuGet API-Key Realm` active and hit save.
![Activate NuGet API-Key Realm](/images/2017/04/Nexus-Add-Nuget-Realm.png)

7) Ensure anonymous access is enabled by going to `Anonymous` in the left hand navigation and ticking the box, and hitting save. **(You will need to do this even if you're planning to use an API key to *consume* modules because when NuGet uploads your package it first reads the repository anonymously to see if the package version already exists.)**
![Nexus Allow Anonymous Access](/images/2017/04/Nexus-Allow-Anonymous-Access.png)

8) Get your NuGet API key by clicking on your username in the top right hand corner (i.e. Admin), then clicking "NuGet API Key" on the left, and "Access Key", entering your password when prompted. Make a note of this, you'll be using it later!
![Access Nuget API Key](/images/2017/04/Nexus-Get-Nuget-API-Key.png)

# Configuring PowerShell
## Register your new repository with the following
```
Register-PSRepository -Name MyCustomRepo -SourceLocation http://yourserver:8081/repository/powershell-modules/ -PublishLocation http://yourserver:8081/repository/powershell-modules/ -PackageManagementProvider nuget -InstallationPolicy Trusted
```
(All one line, [splat](https://msdn.microsoft.com/en-us/powershell/reference/5.1/microsoft.powershell.core/about/about_splatting) it if you like!)  
## Publish Your Module
Ensure you have a Module Manifest, if not, create one using [New-ModuleManifest](https://msdn.microsoft.com/en-us/powershell/reference/5.1/microsoft.powershell.core/new-modulemanifest).  
Also ensure you have the following properties filled out: 

* RootModule
* ModuleVersion
* GUID
* Author
* CompanyName
* Copyright
* Description  

Or it *will not* publish successfully.
In a PowerShell console CD into the directory that has your module and run the following:
```
Publish-Module -Name .\MyModule.psd1 -Repository MyCustomRepo -Verbose -NuGetApiKey "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```
Providing you were successful (if not, see [Troubleshooting](#Troubleshooting)), congrats! You have now successfully uploaded your 
# Listing Modules
`Find-Module -Repository MyCustomRepo`
# Installing Modules
`Install-Module -Name MyModule -Repository MyCustomRepo`

# Updating Modules
This is where the `ModuleVersion` becomes really important. You *cannot* overwrite an existing module version with new code. If you try it, you'll get the following error:

```
Publish-Module : The module 'MyModule' with version '1.0' cannot be published as the current version '1.0' is already available in the repository 
'http://yourserver:8081/repository/powershell-modules/'.
At line:1 char:1
+ Publish-Module -Name .\MyModule.psd1 -Repository MyCustomRepo  -Verbo ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (:) [Publish-Module], InvalidOperationException
    + FullyQualifiedErrorId : ModuleVersionIsAlreadyAvailableInTheGallery,Publish-Module
```

To avoid this, just increment the version in the Module Manifest (`psd1` file), and retry.  

When you want to download the latest version I would actually recommend
`Install-Module -Name MyModule -Repository MyCustomRepo -Force`  
As this immediately imports the new version, where `Update-Module` does not.

# Troubleshooting
If you get the following error when trying to publish your module, you've either  

1. Forgotten to add `NuGet API-Key Realm` to the active realms in Nexus
2. Forgotten to enable anonymous access in Nexus

```
Publish-PSArtifactUtility : Failed to publish module 'MyModule': 'Cannot prompt for input in non-interactive mode.
'.
At C:\Program Files\WindowsPowerShell\Modules\PowerShellGet\1.1.2.0\PSModule.psm1:1227 char:17
+                 Publish-PSArtifactUtility -PSModuleInfo $moduleInfo `
+                 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (:) [Write-Error], WriteErrorException
    + FullyQualifiedErrorId : FailedToPublishTheModule,Publish-PSArtifactUtility
```
If you get the following error when trying to publish, you forgot to include the `-PublishLocation` argument when running `Register-PSRepository`. You'll need to `Unregister-PSRepository` before re-running it.

```
Publish-Module : The specified repository 'MyCustomRepo' does not have a valid PublishLocation. Retry after setting the PublishLocation for repository 'MyCustomRepo' to a valid NuGet 
publishing endpoint using the Set-PSRepository cmdlet.
At line:1 char:1
+ Publish-Module -Name .\MyModule.psd1 -Repository MyCustomRepo  -Verbo ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidArgument: (MyCustomRepo:String) [Publish-Module], ArgumentException
    + FullyQualifiedErrorId : PSGalleryPublishLocationIsMissing,Publish-Module
```

