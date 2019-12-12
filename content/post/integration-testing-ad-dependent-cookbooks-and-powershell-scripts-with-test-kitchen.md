+++
author = "toukakoukan"
categories = ["Chef", "powershell", "test kitchen"]
date = 2016-02-22T00:00:00Z
description = ""
draft = false
slug = "integration-testing-ad-dependent-cookbooks-and-powershell-scripts-with-test-kitchen"
tags = ["Chef", "powershell", "test kitchen"]
title = "Integration Testing AD Dependent Cookbooks and PowerShell Scripts with Test-Kitchen"

+++

I remember a little while ago, well… over a couple of years ago now… that I’d just learned about Chef and Vagrant, and was introducing one of my co-workers to the ecosystem (somewhat over-enthusiastically), and wanted to help him perform his immediate deployment task using Chef. That deployment task happened to be to deploy and configure a Remote Desktop Gateway. So we spun up a Windows 2012R2 Vagrant box, use the Windows cookbook to add the appropriate Windows feature, ran it, and **bam**, brick wall. In order to install the feature the server must be a member of an AD domain. At the time I had to abandon the endeavour and relent that my co-worker had to install the system manually as I didn’t have a good story for creating a Vagrant test environment joined to a domain. But no more!


## Introducing test\_kitchen\_ad\_helpers

[![Screenshot from 2016-03-20 16-23-42](/images/2016/02/Screenshot-from-2016-03-20-16-23-42.png)](/images/2016/02/Screenshot-from-2016-03-20-16-23-42.png)

So what does it do? First let’s briefly explain [Test Kitchen](http://kitchen.ci/). Test Kitchen is a wrapper on top of Vagrant which manages the process of downloading dependent cookbooks, spinning up instances, provisioning them in accordance with your definition, and running integration tests against them using a variety of testing frameworks like [ServerSpec](http://serverspec.org/), [Pester](https://github.com/test-kitchen/kitchen-pester), [Bats](https://github.com/sstephenson/bats), etc. With one command you can download the vagrant box, spin up a variety of different OSes, run your cookbook against them, then validate they worked as expected.

How does this relate to AD though? Glad you asked. If, like in the example at the beginning of this post, you’re looking to create a cookbook that depends on an AD domain to run (or you want to test how a cookbook that should be AD independent runs on a member server), then usually you need to break your typical testing workflow and go and either run your cookbook in a pre-existing domain-joined test environment, or manually spin up a Vagrant box, dcpromo it, join another box to the domain, blah blah blah. Very time consuming and now, unnecessary!

Using [kitchen-nodes](https://github.com/mwrock/kitchen-nodes) ([@MattWrock](https://twitter.com/mwrockx)‘s test kitchen provisioner which allows you to use Chef Search inside test-kitchen), and [windows\_ad ](https://github.com/TAMUArch/cookbook.windows_ad) ([Texas A&M University, College of Architecture](http://www.arch.tamu.edu/)‘s AD cookbook) [test\_kitchen\_ad\_helpers](https://github.com/Sam-Martin/test_kitchen_ad_helpers) allows you to provision an Active Directory domain controller prior to spinning up your test instance, and then join that test instance to the test domain automatically.


## How does it work?

Two nodes are defined as platforms and provisioned under the default suite in Test Kitchen, *windows-domaincontroller *and *windows-2012r2*, both running [mwrock/Windows2012R2](https://atlas.hashicorp.com/mwrock/boxes/Windows2012R2) because it’s the de-facto 2012r2 Vagrant box in my mind. Both boxes are set to have Private Virtualbox NICs with DHCP assigned IPs. The former is set to run *[test\_kitchen\_ad\_helpers::server](https://github.com/Sam-Martin/test_kitchen_ad_helpers/blob/master/recipes/server.rb)* (a wrapper recipe that installs windows features using PowerShell so that we can download the ones not bundled in the Vagrant box). The latter runs *[test\_kitchen\_ad\_helpers::client](https://github.com/Sam-Martin/test_kitchen_ad_helpers/blob/master/recipes/client.rb)* which identifies the IP of the domain controller using Chef Search (enabled by [kitchen-nodes](https://github.com/mwrock/kitchen-nodes)), sets the private NIC’s DNS server to that IP, then joins the node to the test domain using the windows_ad cookbook.

Pretty simple really, but having it as a template allows you to just consume it as a made and tested pattern rather than having to put it together and test it yourself. Speaking of which!


## How do I use it?

You only need two files [from the repository](https://github.com/Sam-Martin/test_kitchen_ad_helpers).

1. .kitchen.yml
2. Berksfile

In the .kitchen.yml you don’t really want to manipulate the *windows-domaincontroller *node, as that’s what’s giving you your domain, you want to run your cookbooks and tests against the *windows-2012r2* node.

If you define new test-suites, make sure to exclude the *windows-domaincontroller* node using the *exclude:* suite property from here: [https://github.com/test-kitchen/test-kitchen/issues/29](https://github.com/test-kitchen/test-kitchen/issues/29).

If you want to add additional member-server nodes, make sure to copy the following property from the *windows-2012r2* node so it joins the domain:

` run_list: - test\_kitchen\_ad\_helpers::server`

The Berksfile you can just merge with your existing Berksfile (if you have one).

The only other non-GitHub-related files in the repo relate to testing the setup in my repository. Obviously you don’t really need to test the servers join the domain, that’s the responsibility of the pattern I’m providing, but you can if you like by copying *[default.spec](https://github.com/Sam-Martin/test_kitchen_ad_template/blob/master/test/integration/default/serverspec/default_spec.rb).*


## A warning

Don’t use **kitchen test**. The **kitchen test** command spins up one node, converges it, tests it, then destroys it.

This is great when nodes don’t have interdependencies, but is no use when the second node depends on the first!

Instead you need to run:

1. **kitchen converge**
2. **kitchen verify**
3. **kitchen destroy**

This prevents you from accidentally destroying the AD domain controller your second node depends on!


## Further Reading

This template is really a trivial wrapper on top of Matt Wrock’s [kitchen-nodes](https://github.com/mwrock/kitchen-nodes) so I thoroughly recommend you read his blog posts on multi-node testing with Chef Kitchen.

- [Orchestrating multi node Windows tests in Test-Kitchen Beta!](http://www.hurryupandwait.io/blog/orchestrating-multi-server-tests-in-test-kitchen)
- [Multi node testing with Test-Kitchen and Docker containers](http://www.hurryupandwait.io/blog/multi-node-testing-with-test-kitchen-and-docker)
- [Multi node Test-Kitchen tests and working with Vagrant NAT addressing with VirtualBox](http://www.hurryupandwait.io/blog/multi-node-test-kitchen-tests-and-working-with-vagrant-nat-addressing-with-virtualbox)

