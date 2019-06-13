+++
author = "toukakoukan"
date = 2013-02-09T00:00:00Z
description = ""
draft = false
slug = "scripting-backup-exec-2012-p3-customising-and-adding-backup-tasks"
title = "Scripting Backup Exec 2012 - P3 - Customising and Adding Backup Tasks"

+++

# Introduction

Part 3 will cover creating and customising backup tasks for our newly created backup definition. So far we’ve only accepted the defaults, so unless you’re happy with the predefined schedule, retention period and secondary backup type, you’re going to have to start customising your tasks!


# Customising and Adding Backup Tasks

As we’ve [already created](http://wp.me/pBnHz-43) a backup definition from the BackupToDisk defaults, we’ll have a new definition  containing a full backup and an incremental backup.

It’s important to note that a backup definition will *always* contain a full backup tasks, you cannot delete it. You can only ever modify it. And in order to do so, we must first obtain our definition as a variable. There are two options for doing this, either you can assign your new definition to a variable as you create it, like so:

```
$backupDefinition = New-BEBackupDefinition -BackupJobDefault BackupToDisk -FileSystemSelection $selection -SystemStateSelection Excluded -name "New Backup Definition" -AgentServer $agentServer
```

Or, you can retrieve one you made earlier using:

```
$backupDefinition = get-BEBackupDefinition -name "New Backup Definition"
```

**Gotcha:** In theory, these two are interchangeable, but on a number of occasions I found that using the second method (`Get-BEBackupDefinition`) immediately after saving or creating a new backup definition, caused the cmdlet to complain it couldn’t find a definition by that name. So on the whole I would recommend the former over the latter.

## Cleaning Up and Scheduling

As I’ve already mentioned, you can only ever customise the full backup task, but as we also have an incremental task in our definition, let’s delete that before we get started.

```
$backupDefinition = $($backupDefinition | Remove-BEBackupTask * -Confirm:$false | Save-BEBackupDefinition -confirm:$false | Get-BEBackupDefinition)
```

In this section of code, we’re piping the $backupDefinition variable to `Remove-BEBackupTask`, which is using a wildcard (“*”) to delete all backup tasks (except full as you can’t delete that!). This then pipes the modified backup definition to the `Save-BeBackupDefinition` cmdlet, which commits the change.

**Gotcha:** Note the $() around the piped commands, without this, you won’t get the modified definition back, just the output of the first pipe.

**Double Gotcha:** Since writing this post I’ve discovered that occasionally the definition would not be passed correctly to the variable, causing subsequent cmdlets that I passed it to to claim that it was null. This is why I’ve put the `| Get-BEBackupDefinition` at the end, as this, oddly, seems to resolve the problem.

Now that we’ve cleared out the cobwebs, let’s see what we need in order to customise a task:

- Schedule
- Storage
- Retention Period

The schedule we create using:

```
$fullBackupSchedule = New-BESchedule -WeeklyEvery "Saturday" -startingAt "8am"
```

We’re going to use this to modify our full backup to run weekly on a Saturday at 8am. There are, naturally, a huge range of options in the `New-BESchedule` cmdlet, I recommend you consult the help file mentioned in [Part 1](http://wp.me/pBnHz-2p).

**Gotcha:** Where the help file says you can specify multiple days by comma separating them, you cannot. You must insert them into an array first like so:

```
[...] New-BESchedule -WeeklyEvery @("Friday", "Saturday", "Sunday") [...]
```

# Modifying and Creating Tasks

The schedule is the only tricky part of modifying or creating a backup task. For this purpose there are six commands you need to know:

- Set-BEFullBackupTask
- Set-BEIncrementalBackupTask
- Set-BEDifferentialBackupTask
- Add-BEFullBackupTask
- Add-BEIncrementalBackupTask
- Add-BEDifferentialBackupTask

You can probably guess, set is for modify, add  is for creating new ones.

So, first up is modifying our existing task using the `$schedule` we created earlier.

```
$backupDefinition = $(Set-BEFullBackupTask -backupDefinition $backupDefinition  -Name * -DiskStorageKeepForHours 672 -schedule $fullBackupSchedule -storage "Full Backup Storage Pool" | Save-BEBackupDefinition -confirm:$false)
```

This passes the existing backup definition from the `$backupDefinition` variable, using the asterisk wildcard to set ALL full backup tasks for this definition (there’s only one) to keep the storage for 4 weeks (672 hours) according to our previously defined schedule in `$fullBackupSchedule` and to backup to the “Full Backup Storage Pool”. It then pipes the modified definition to the `Save-BEBackupDefinition` cmdlet which saves the change. All this is encapsulated by $() which ensures all the piped commands are executed fully prior to saving the output (the newly altered and committed definition) into `$backupDefinition`.


# The Script So Far

Phew! So far we’ve got a script that creates a new backup definition, deletes the incremental default and modifies the full backup task to our specifications. So what does it look like in full?

```
# Select the correct server
$agentServer = getBeAgentServer "server11*"

# Include the following directories
$selection= @(New-BEFileSystemSelection -path "C:\*" -PathIsDirectory -Recurse)
$selection = New-BEFileSystemSelection -path "Z:\* -PathIsDirectory -Recurse

# Exclude the following directories
$selection +=&nbsp;New-BEFileSystemSelection -path "C:\Windows\* -PathIsDirectory -Exclude

# Create the Backup definition based on defaults
New-BEBackupDefinition -BackupJobDefault BackupToDisk&nbsp;-FileSystemSelection $selection -SystemStateSelection Excluded -name "New Backup Definition" -AgentServer $agentServer

# Delete the default incremental
$backupDefinition = $($backupDefinition | Remove-BEBackupTask * -Confirm:$false | Save-BEBackupDefinition -confirm:$false)

# Create a full backup schedule
$fullBackupSchedule = New-BESchedule -WeeklyEvery "Saturday" -startingAt "8am"&nbsp;

# Modify the default full backup to reflect our new schedule and retention period
$backupDefinition = $(Set-BEFullBackupTask -backupDefinition $backupDefinition -Name * -DiskStorageKeepForHours 672 -schedule $fullBackupSchedule -storage "Full Backup Storage Pool" | Save-BEBackupDefinition -confirm:$false)

# Create a differential backup schedule
$differentialBackupSchedule = New-BESchedule -WeeklyEvery @("Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday") -startingAt "9pm"&nbsp;

# Modify the default differential backup to reflect our new schedule and retention period
$backupDefinition = $(Add-BEDifferentialBackupTask -backupDefinition $backupDefinition &nbsp;-Name * -DiskStorageKeepForHours&nbsp;168 -schedule $differentialBackupSchedule&nbsp;-storage "Differential Backup Storage Pool" | Save-BEBackupDefinition -confirm:$false)
```

As you can see, I added a new differential backup task running every day that the full doesn’t run, at 9pm with a retention period of 1 week. Hopefully from the explication of the previous cmdlets you can figure out how to customise it to suit your tastes.


# Future Posts

That’s it for now! I will shortly post two new entries on Backup Exec 2012 on the following topics:

- Integrating Backup Exec with PRTG Monitoring
- Creating a more robust job creation script

Take care, and don’t forget to leave a comment!

