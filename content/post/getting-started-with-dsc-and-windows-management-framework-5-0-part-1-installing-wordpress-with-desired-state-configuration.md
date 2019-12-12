+++
author = "toukakoukan"
date = 2014-09-08T00:00:00Z
description = ""
draft = false
image = "/images/2014/09/wmf-5-0-september-2014-preview.png"
slug = "getting-started-with-dsc-and-windows-management-framework-5-0-part-1-installing-wordpress-with-desired-state-configuration"
title = "Getting Started with DSC and PowerShell 5.0 - Part 1 - Installing WordPress with Desired State Configuration"

+++

So we’ve checked out the basics of Chef on Windows in [Part 1](http://sammart.in/2014/08/24/getting-started-with-chef-on-windows-server/) and [Part 2](http://sammart.in/2014/08/25/getting-started-with-chef-on-windows-server-part-2-chef-server-bootstrapping/) of Chef On Windows, and with the recent release of the [Windows Management Framework 5.0 Preview September 2014 ](http://www.microsoft.com/en-us/download/details.aspx?id=44070) I thought it was time to stick a toe into the water of the Desired State Configuration side of configuration management on Windows.  
 As quite a lot of intros focus very heavily on the theory and don’t necessarily show a lot of results up front, I’m going to continue the precedent of the preview Chef articles and show you the shortest path to something tangible, hopefully gaining some familiarity with the tech involved along the way.

In Part 1 we’re going to use the WMF 5.0 preview, DSC, and a little bit of [OneGet](http://blogs.technet.com/b/windowsserver/archive/2014/04/03/windows-management-framework-v5-preview.aspx)/[PowerShellGet](http://blogs.msdn.com/b/powershell/archive/2014/05/14/windows-management-framework-5-0-preview-may-2014-is-now-available.aspx) (name seems to be up for discussion the moment), to install [WordPress 4.0 ](https://wordpress.org/news/2014/09/benny/)on to a blank VM. In order to do this we’re going to follow the guide laid out in the quick-start of the [WordPress PowerShell/DSC module](http://gallery.technet.microsoft.com/scriptcenter/xWordPress-Module-5d007ff9), so all credit goes to the wonderful people who created this module for providing our first entry point into DSC!

**Important:** You don’t need WMF 5.0 to use DSC, it’s been around since PS 4.0, but the [WordPress PowerShell/DSC module](http://gallery.technet.microsoft.com/scriptcenter/xWordPress-Module-5d007ff9) we’ll be using requires WMF 5.0 for OneGet.

**Important #2:** This guide uses a WMF 5.0 *preview* and DSC modules that are labelled *x* for eXperimental, don’t use these in production ![:)](https://sammart.in/wp-includes/images/smilies/simple-smile.png)


# Requirements

1. Blank [Windows 2012 R2](http://technet.microsoft.com/en-gb/evalcenter/dn205286.aspx) [VM ](https://www.virtualbox.org/)
2. Powershell Understanding &#128;&#147; [Basic: Microsoft Virtual Academy &#128;&#147; Getting Started With PowerShell](http://www.microsoftvirtualacademy.com/training-courses/getting-started-with-powershell-3-0-jump-start)

We won’t need the VMs we created in the Chef series as we’ll be focussing on just DSC for today.


# 1) Preparing the VM

As the  [WordPress PowerShell/DSC module](http://gallery.technet.microsoft.com/scriptcenter/xWordPress-Module-5d007ff9) we’ll be using requires WMF 5.0 for OneGet, we need to go and grab the September 2014 Preview!

Download WMF 5.0 to your 2012 VM from [http://www.microsoft.com/en-us/download/details.aspx?id=44070](http://www.microsoft.com/en-us/download/details.aspx?id=44070) [![WMF 5.0 September 2014 Preview](/images/2014/09/wmf-5-0-september-2014-preview.png)](/images/2014/09/wmf-5-0-september-2014-preview.png)

Now we need to install the [xWordPress module](http://gallery.technet.microsoft.com/scriptcenter/xWordPress-Module-5d007ff9) and its dependencies.

Whoa whoa whoa, don’t download it from the link! What is this, the 90s? We’ve *just * installed PowerShell 5.0 and with it, OneGet, let’s use it!

Open up a PowerShell console and run
```
Install-Module xWebAdministration -MinimumVersion 1.3.2 -Force
```
and accept the offer to download NuGet_anycpu.exe.

[![install-module xWebAdministration](/images/2014/09/install-module-xwebadministration.png)](/images/2014/09/install-module-xwebadministration.png)

Now install the remaining modules.

Install-Module xPSDesiredStateConfiguration -MinimumVersion 3.0.1 -Force Install-Module xMySql -MinimumVersion 1.0 -Force Install-Module xWordPress -MinimumVersion 1.0 -Force Install-Module xPhp -MinimumVersion 1.0.1 -Force

Excellent! Okay, where did they go?
```
$env:ProgramFiles\WindowsPowerShell\Modules folder
```
[![Program Files WindowsPowerShell Modules](/images/2014/09/program-files-windowspowershell-modules.png)](/images/2014/09/program-files-windowspowershell-modules.png)

Awesome! Since when has that been a thing? WMF 5.0? I assume, but I’m not sure. Getting modules to load automatically has always been a bit of a per-user PITA in the past, so if this is user-agnostic way of installing PowerShell modules, it’s only a good thing!


# 2) Prepare the Configuration

Now we need to grab the sample files from the xWordPress module and customise them to our needs.

Copy the contents of `C:\Program Files\WindowsPowerShellModules\xWordPresssamples` to your Documents folder

[![samples in my documents](/images/2014/09/samples-in-my-documents.png)](/images/2014/09/samples-in-my-documents.png)

Open up `SingleNodeEndToEndWordPress.ps1` in the PowerShell ISE and check that the Download URLs are still correct for PHP and MySQL.

[![MySQL and PHP URLs](/images/2014/09/mysql-and-php-urls.png)](/images/2014/09/mysql-and-php-urls.png)

I only had to change PHP to [http://windows.php.net/downloads/releases/archives/php-5.5.14-nts-Win32-VC11-x64.zip](http://windows.php.net/downloads/releases/archives/php-5.5.14-nts-Win32-VC11-x64.zip), but double check MySQL as well, as it may have changed by the time you read this!


# 3) Executing the Configuration

Go back to your PowerShell window, cd into your documents folder and execute `SingleNodeEndToEndWordPress.ps1`.

This will perform the following tasks (at least):

1. Install IIS
2. Install PHP and dependencies
3. Install MySQL
4. Install WordPress into IIS with * port 80 HTTP bindings.

[![SingleNodeEndToEndWordPress](/images/2014/09/singlenodeendtoendwordpress.png)](/images/2014/09/singlenodeendtoendwordpress.png)

After some time, your system will restart to complete the installation.

[![DSC is Restarting the computer](/images/2014/09/dsc-is-restarting-the-computer.png)](/images/2014/09/dsc-is-restarting-the-computer.png)

Once it’s restarted, DSC will continue to configure the computer, to see the progress, go to the DSC event log.  (Event Viewer > Applications and Services Log > Microsoft > Windows > Desired State Configuration > Operational)![DSC Event Log](/images/2014/09/dsc-event-log.png)

Once you see “Warning” “The local configuration manager was shut down”, your new WordPress site should be ready! Check out Localhost in IE!

[![WordPress 4.0 default](/images/2014/09/wordpress-4-0-default.png)](/images/2014/09/wordpress-4-0-default.png)

Ooh, this is the first time I’ve seen WordPress 4.0 default installation! First impressions are very monochrome, but eh, that’s what themes are for!


# Summary

So what have we achieved here?

We’ve used community provided modules for DSC/PowerShell to install WordPress and *all* its dependencies, including IIS, PHP, and MySQL.

Was this easier than doing all the work ourselves, clicking through installers and typing out config ourselves? Much!

Does it mean we no longer need Chef and all that work we did in the past couple of posts was unnecessary? Not at all!

Does this illustrate the power and flexibility of DSC and OneGet? No, we’re just getting started!

I’ll be writing a subsequent post to dig in and write our own DSC module/template/whatever-the-correct-nomenclature–is but I suspect that bringing what we’ve learned today into Chef with [Chef’s new DSC evaluation release recipes](http://www.getchef.com/blog/2014/07/24/getting-ready-for-chef-powershell-dsc/) will be the post immediately following this one.


# Further Reading

Steven Murawski

- [GitHub PowerShell Community DSC Modules](https://github.com/powershellorg/dsc)
- [Building a Desired State Configuration Infrastructure](http://powershell.org/wp/2013/10/02/building-a-desired-state-configuration-infrastructure/)- [Building a Desired State Configuration Configuration](http://powershell.org/wp/2013/10/08/building-a-desired-state-configuration-configuration/)
- [Building a Desired State Configuration Configuration &#128;&#147; Part 2](http://powershell.org/wp/2013/10/14/building-a-desired-state-configuration-configuration-part-2/)
- [Building Desired State Configuration Custom Resources](http://powershell.org/wp/2014/03/13/building-desired-state-configuration-custom-resources/)

Everything Else

- [xWordPress Module &#128;&#147; PowerShell Desired State Configuration Resource Kit](http://gallery.technet.microsoft.com/scriptcenter/xWordPress-Module-5d007ff9)
- [Checking Out OneGet in PowerShell V5](http://learn-powershell.net/2014/04/03/checking-out-oneget-in-powershell-v5/)
- [Getting Started with Chef on Windows Server &#128;&#147; Part 1 Intro](http://sammart.in/2014/08/24/getting-started-with-chef-on-windows-server/)
- [Getting Started with Chef on Windows Server &#128;&#147; Part 2 &#128;&#147; Chef Server & Bootstrapping](http://sammart.in/2014/08/25/getting-started-with-chef-on-windows-server-part-2-chef-server-bootstrapping/)

