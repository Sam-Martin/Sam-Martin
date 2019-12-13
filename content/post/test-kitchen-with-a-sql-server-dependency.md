+++
author = "Sam Martin"
categories = ["Chef", "ms sql", "sql server", "test kitchen", "Windows"]
date = 2016-03-04T00:00:00Z
description = ""
draft = false
image = "/images/2016/03/Screenshot-from-2016-03-20-16-19-09.png"
slug = "test-kitchen-with-a-sql-server-dependency"
tags = ["Chef", "ms sql", "sql server", "test kitchen", "Windows"]
title = "Test Kitchen with a SQL Server Dependency"
aliases = ['/test-kitchen-with-a-sql-server-dependency/']
+++

I want to be able to test my cookbooks/scripts without having to rely on independent testing infrastructure! I want to be able to do *kitchen converge* and have all my infrastructure ready to go without any further thought to the dependencies!

This is why I’m such a huge fan of Matt Wrock’s [Kitchen-Nodes](https://github.com/mwrock/kitchen-nodes). It’s a provisioner wrapper around Chef-Zero for Test Kitchen, which allows you to access some of the attributes from other nodes which you’re provisioning in Test Kitchen using Chef Search.

I was hoping to be able to init kitchen, grab the official MS SQL cookbook, and be on my way.

Ha, ha, no. Want to install SQL server using remotely via WinRM as is required to provision it using Test-Kitchen?

### You Goddamned Can’t.

See [here](http://stackoverflow.com/questions/26523301/powershell-remoting-executing-sql-server-installation-msi-fails) comments [here](https://learn.chef.io/manage-a-web-app/windows/configure-sql-server/), and of course, on [Matt Wrock’s blog](http://www.hurryupandwait.io/blog/safely-running-windows-automation-operations-that-typically-fail-over-winrm-or-powershell-remoting).

I’m sure this isn’t news to some of you, I came across this limitation in Windows in the with respect to [remotely installing Windows Updates](http://serverfault.com/questions/336705/issues-with-patching-servers-remotely-using-winrm-and-microsoft-update-session). Fact is, there are some things you just can’t do from within the context of a remote session in Windows.

Upside is, you can initiate the execution of your scripts in a local context *from* a remote context using… Scheduled Tasks!
```
$STPrin = New-ScheduledTaskPrincipal -UserId "SYSTEM" -LogonType ServiceAccount 
$STTri = New-ScheduledTaskTrigger -Once -At $(Get-Date) -RepetitionInterval "00:01:00" -RepetitionDuration $([TimeSpan]::MaxValue) 
$STAct = New-ScheduledTaskAction -Execute "PowerShell.exe" ` 
   -Argument $('-executionpolicy Bypass -NonInteractive -c "powershell -executionpolicy Bypass -NonInteractive -c '+$($MyInvocation.MyCommand.Definition)+' -verbose >> '+$LogLocation+'.scheduledtask 2>&1"') 
Register-ScheduledTask -Principal $STPrin -Trigger $STTri -TaskName $TaskName -Action $STAct
```
Blah, boring, repeatable (also this is an old example and I didn’t use [splatting](https://technet.microsoft.com/en-us/magazine/gg675931.aspx), oops).

Enter our remote-execution saviour!


## [Boxstarter!](http://boxstarter.org/)

[![2016-02-16 - Boxstarter](/images/2016/02/2016-02-16-Boxstarter.png)](/images/2016/02/2016-02-16-Boxstarter.png)

Wait, [Matt Wrock](http://hurryupandwait.io) made this? Wow, it seems like every problem I encounter recently that’s vaguely related to Chef on Windows has already been solved by him. Thanks Matt!

Boxstarter is a series of PowerShell cmdlets that wrap around [Chocolatey](https://chocolatey.org/) to allow you to easily manage the installation of a Chocolatey package remotely or locally, whether or not Chocolatey is already installed.

Particularly useful for us is the [Boxstarter cookbook](https://github.com/mwrock/boxstarter-cookbook), which I’ve used to install the [SQL Express 2014 Chocolatey package](https://chocolatey.org/packages/MsSqlServerManagementStudio2014Express) on a node as part of a [helper cookbook](https://github.com/Sam-Martin/test_kitchen_mssql_helpers).


## test\_kitchen\_mssql\_helpers

[![Screenshot from 2016-03-20 16-19-09](/images/2016/03/Screenshot-from-2016-03-20-16-19-09.png)](/images/2016/03/Screenshot-from-2016-03-20-16-19-09.png)

Right. Yes! Testing MSSQL dependent cookbooks using Test Kitchen, that’s why we’re here!

So what does this do? It spins up two nodes using test-kitchen:

1. windows-mssql
2. windows-2012r2

The star of the show is the first node, which is setup to install the [SQL Express 2014 Chocolatey package](https://chocolatey.org/packages/MsSqlServerManagementStudio2014Express) using the [Boxstarter cookbook](https://github.com/mwrock/boxstarter-cookbook) to get around any niggling WinRM issues.

As part of the [wrapper cookbook’s server recipe](https://github.com/Sam-Martin/test_kitchen_mssql_helpers/blob/master/recipes/server.rb), the windows-mssql node is also setup to listen on port 1433 on all IPs, and to use mixed-mode auth with sa setup to use Vagrant! as its password. Secure!

This first node is setup first, and is thereby accessible on the nodes’ private VirtualBox network when the other node (windows-2012r2) provisions itself.

Without kitchen-nodes, figuring out the IP of the SQL server from within the context of the second node would be damn-near impossible. Fortunately **with** kitchen-nodes, it’s just a simple Chef Search away!
```
search_query = 'run_list:*test_kitchen_mssql_helpers??server*' sql_server_ip = search('node', search_query)[0]['ipaddress']
```
Or you can see the [ServerSpec test which runs on each node](https://github.com/Sam-Martin/test_kitchen_mssql_template/blob/master/test/integration/default/serverspec/default_spec.rb) as an example of how to parse the JSON file kitchen-nodes propagates into the nodes and then connect to it using the TinyTDS Ruby gem.


## So how do I use it?

You only need two files from the [test_kitchen_mssql_helpers repo](https://github.com/Sam-Martin/test_kitchen_mssql_helpers).

1. .kitchen.yml
2. Berksfile

Use .kitchen.yml as the basis for your kitchen tests adding your cookbook to the windows-2012r2 run list, and adding your ServerSpec, Rspec, whatever, tests into .*/test/integration/default/<insert-test-framework-name-here>.*


## A warning

Don’t use **kitchen test**. The **kitchen test** command spins up one node, converges it, tests it, then destroys it.

This is great when nodes don’t have interdependencies, but is no use when the second node depends on the first!

Instead you need to run:

1. **kitchen converge**
2. **kitchen verify**
3. **kitchen destroy**

This prevents you from accidentally destroying the MSSQL server your second node depends on!

