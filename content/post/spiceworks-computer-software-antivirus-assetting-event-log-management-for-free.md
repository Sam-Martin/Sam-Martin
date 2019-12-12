+++
author = "toukakoukan"
date = 2011-04-14T00:00:00Z
image = "/images/2011/04/spiceworks.jpg"
description = ""
draft = false
slug = "spiceworks-computer-software-antivirus-assetting-event-log-management-for-free"
title = "Spiceworks Computer, Software & Antivirus Assetting + Event Log Management. For.. Free?!"

+++

Nearly 7 years ago, at the tender age of 18 I spent about 6 months coding and perfecting a VBScript/WMI software & computer assetting tool for Sysmex UK, the company I was working for at the time.  
 I was coding it myself because the asset management systems at the time were either incredibly poorly featured, labour intensive, expensive or all three.

I would have given my right arm for something along the lines of [SpiceWorks ](http://www.spiceworks.com "Spiceworks - Software and Computer Management")

[![](/images//2011/04/spiceworks.jpg?w=300 "Spiceworks")](/images//2011/04/spiceworks.jpg)

[](/images//2011/04/spiceworks.jpg)It consists of a web-interface hosted by a discreet Apache server, from which you can configure & view:

- Network Range
- Active Directory Users
- Computers
- Installed Software
- Antivirus Installs & Update Status
- Event Logs

All of which is absolutely essential in an enterprise business environment; which is why corporations pay tens of thousands for such systems.

But what about the small to medium businesses? Those that need to asset their software and hardware, but who can’t justify thousands of pounds to manage 5-20 computers?  
 Well, that’s where the best thing about Spiceworks comes  in, it’s **FREE**.  
 Ideally they need to have an AD environment of some description or else you’ll have to make sure there’s a local account on each computer setup with the same username and password that has WMI rights.

I was rather worried about the security implications at first, but as far as I’ve been able to discover, they’re a very reputable company who seem to take their customer’s privacy very seriously.  
 In *theory* it should even comply with ISO27001 as none of the data is hosted off-site, but don’t quote me on it.

Anyway, the point of this post was to draw attentions to some difficulties I had while configuring it initially.


# My Initial Configuration Problems


## Computers Not Appearing Inventory after Network Scan

I had this problem initially and quickly discovered it was due to ICMP (ping) being blocked by the client PCs firewalls.  
 You can change this via group policy by enabling it under the following:

`Computer Configuration > Administrative Templates > Network > NetworkConnections > Windows Firewall > Domain Profile`


## Active Directory Users Not Importing (Empty People Inventory)

This could be caused by a number of things:

Most common apparently seems to be the username you’re entering to query AD with is formatted incorrectly.  
 It should be *username@domain*. e.g. *administrator@contoso.com*

Alternatively it could be that the base DN needs configuring.  
 In  Spiceworks go to

` Settings > Active Directory > Pro Settings`

And configure the Base DN to the OU that the users you want to enumerate are in e.g.

` OU=Staff,DC=contoso,DC=com`

**In my case** it was that my users *didn’t have email addresses*.  
 You can fix this either by manually adding email addresses  through AD itself, or going to Pro Settings in Spiceworks and changing

`Use principal name for email if blank in Active Directory`

To ‘true’.

 

After I’d resolved this initial niggles I fell in love with Spiceworks! Thanks guys!

