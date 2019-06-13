+++
author = "toukakoukan"
date = 2013-02-09T00:00:00Z
description = ""
draft = false
slug = "scripting-backup-exec-2012-p1-introduction-references-gotchas"
title = "Scripting Backup Exec 2012 - P1 - Introduction, References & Gotchas"

+++

# Preamble


## Introduction

Over the past week, I’ve been trialling and implementing Backup Exec 2012 to replace an existing Backup Exec 2010 setup. Although Googling for Backup Exec 2012 reviews gets you a great deal of negativity and vitriol about the new interface, I have to say that I find it far more intuitive than the previous system. Where previously even something as simple as adding a backup client to the server required some headscratching and occasionally some swearing, Backup Exec 2012 now centres the interface around common tasks, rather than a semi-arbitrary architecture that probably made a great deal of sense to the programmers.

That isn’t to say it’s all perfect, the decision to have graphically intensive screen animations for a utility that’s going to be primarily used over an RDP connection is unfathomable, and the inability to turn them off (as far as I can ascertain) is even more so.The logic behind some of the locations of some of the actions is a little confusing as well. For example, in order to delete a backup job, you must be in the ‘tree view’ of the job page. If you are in the *list* view, it’s greyed out for no apparent reason. My understanding is that this is due to a lack of transparency of the internal workings (which I was just praising) or to put it another way, a disconnect between the internal workings and the way you would expect it to work. Namely “Jobs” in the GUI and “Definitions” and “Tasks” in the back end.


## This Series & the Problem at Hand

Over the next few posts I will be introducing the basic concepts and practicalities of scripting job creation in BE2012 as I understand them. I do not claim to be an expert in either Powershell or Backup Exec, so I am more than open to correction in the comments.

The reason we’re interested in this is due to one of the most complained about changes in BE2012. The fact that agents in a job are far more separate than in 2010. Where previously in 2010 a selection list made it fairly trivial to flick through different clients and select files and folders on an individual basis, this process is much more time consuming in 2012. It requires going into each server individually, hitting an edit button, waiting for the options to load, etc. While this isn’t an *especially* time consuming process for an individual server (call it a 10-20 second overhead), when you get into the hundreds of servers, this becomes incredibly tedious and is largely a waste of time when you have a server estate with largely similar selection requirements.

This post will primarily be an introduction to concepts, some gotchas, and a dollop of praise for 2012.


# Scripting


## Intro

By far the most wonderful thing about Backup Exec 2012 is the fact that its scripting interface is built in PowerShell and it actually *works*. My experience with the command line interface (BEMCMD) of Backup Exec 2010 was nothing short of atrocious. Baffling switches combined with poorly formatted output were enough to put anyone off, but even once you waded through that, it didn’t even work! You would ask it “Restore W:\FTPsite\* for server XXX” and it would go away for half an hour, think about it, and say “no matches found” even though you had asked it a few moments prior, “What backed up on server XXX recently?” and it replied “W:\FTPSite”.

In this series we will be covering only file backups using the Windows agent, not because DB and Linux backups aren’t important, but because I simply have no experience of scripting them in Backup Exec.


## Concepts

This series is going to primarily revolve around the creation of jobs, though I will do a separate article on PRTG monitoring and may expand to restores and other scripted processes later.

There are four major components to a ‘backup job’:

1. Definition
2. Tasks
3. Schedule
4. Selection

A **definition** is effectively a container for the tasks, it specifies which client we’re backing up and what *selection* we’re backing up.

A **task** is your actual backup job, it specifies what type of backup you’re doing (full, incremental, differential), what *schedule* you’re backing up on, your retention period and your storage device.

A **schedule** is, unsurprisingly, the schedule according to which your *tasks* happen, what days/weeks/months/hours, etc it will run.

A **selection** is, again unsurprisingly, the selection of files that you will be backing up. This is applied to the *definition*.

**GOTCHA:** A *definition* must always have a Full Backup task, you can never delete it, only modify it.


## References

The canonical reference is, of course, the BEMCLI help file, which can be found on the [Symantec Website](http://www.symantec.com/business/support/index?page=content&id=DOC5438 "BEMCLI Help File").

**GOTCHA: **At the time of writing, the .chm file available for download at this link is unviable, its pages are blank.

**GOTCHA UPDATE:** As Roderick Bant points out in the comment (and is now mentioned on the page linked), the above gotcha is untrue.  To get round it: “[…] right-click the file and select **Properties**. Under the **General** tab, you will see a message that says “This file came from another computer and might be blocked to help protect this computer”. Click the **Unblock** button.”

To get around the gotcha, simply copy the same file from your Backup Exec installation directory. I would host it here, but I expect I’d get a cease and desist order fairly quickly from Symantec.

**GOTCHA:** Some of the documentation in the BEMCLI help file is inaccurate. For example, it claims you pass multiple selection directories by simply comma separating them. This is incorrect, you must create them in an array, then pass the array to the cmdlet.

With this in mind, the help file is still an invaluable reference tool, once you learn to take its teachings with a pinch of salt.


## Bugs

The base installation of Backup Exec 2012 I found to be somewhat buggy when dealing with scripts. I would often get SQL locking errors that would cause my jobs to fail to be inserted.

Thankfully, after installing the latest BE2012 hotfixes on the server (which, admittedly I should have done originally), most of these problems went away.


# Next Time…

Next is [Part 2 Scripting Backup Exec 2012 – P2 – Creating a New Backup with Powershell](http://wp.me/pBnHz-43), where we’ll create a basic backup job using the Powershell cmdlets in BE2012.

