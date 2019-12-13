+++
author = "toukakoukan"
date = 2012-09-06T00:00:00Z
description = ""
draft = false
slug = "allow-log-on-through-terminal-services-vs-allow-log-on-through-remote-desktop-services"
title = "Allow log on through Terminal Services vs allow log on through Remote Desktop Services"
aliases = ['/allow-log-on-through-terminal-services-vs-allow-log-on-through-remote-desktop-services/']
+++

Long story short, I was looking for a way to allow a group of users access to specific member servers in active directory.

Much googling brought up various references to “Allow log on through Terminal Services” and “Allow log on through Remote Desktop Services”, mostly references to the former which was not available on my server’s group policy.

After much searching and heart rending, I discovered that these are** both names for the same setting**, which has a machine name of *SeRemoteIn**teractiveL**ogonRight*.

Unfortunately, this setting, despite all the hype, is not what I was after.  
 It works perfectly well for setting user access to:

- Remote Desktop Services Server
- Terminal Services Server

What I wanted, however, was simply to allow the group to log on to the servers the GPO was applied to via the *‘Remote Desktop’ *setting that is available in Advanced System Properties:

![Advanced System Properties](/images/2012/09/remote-desktop-advanced-system-properties1.gif)

The only way I’ve been able to do this via group policy is by adding the group to the “Remote Desktop Users” local group on the servers, using the Restricted Groups policy option which can be found:

* Computer Configuration > Policies > Windows Settings > Security Settings > Restricted Groups*

Right click, ‘Add Group’, and the first group is “BUILTINRemote Desktop Users” (click Okay, don’t browse). Then set the ‘members of this group’ to include the AD security group you wish to have access to log on remotely.

**Important:** Because this is a ‘computer configuration’ policy, you must apply it to an OU that contains the computer accounts that you wish the group to remote desktop into.

Of course, this will not work with domain controllers, because they don’t have local groups, at the time of writing I have not found a way to set this via group policy with domain controllers.


## **The two things that must be set**

1. Remote Desktop connections enabled (as per the above screenshot)
2. The security group added to the local group via Restricted Groups and the instructions above.

Have both of those set and you should be grand.

If you happen to find a way to allow such permissions to be set on Domain Controllers (without installing the Remote Desktop Services role), leave a comment!


#

