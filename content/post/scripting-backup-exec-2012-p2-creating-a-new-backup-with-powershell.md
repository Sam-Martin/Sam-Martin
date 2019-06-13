+++
author = "toukakoukan"
date = 0001-01-01T00:00:00Z
description = ""
draft = false
slug = "scripting-backup-exec-2012-p2-creating-a-new-backup-with-powershell"
title = "Scripting Backup Exec 2012 - P2 - Creating a New Backup with Powershell"

+++

# Introduction

In Part 2 of Scripting Backup Exec 2012, I’ll be illustrating how to create a Windows file backup from scratch using Powershell. For a basic rundown of the components, concepts and gotchas for this process, please see [Part 1](http://wp.me/pBnHz-2p).

**Prerequisites:**

1. A **fully patched and up to date** installation of Backup Exec 2012
2. Powershell and (advantageous) basic Powershell knowledge
3. An [unrestricted execution policy](http://technet.microsoft.com/en-us/library/ee176961.aspx)
4. A pre-installed and trusted Windows Agent added to the Backup Exec server.


# The Script

As mentioned in [Part 1](http://wp.me/pBnHz-2p) to successfully create a backup job we need to programmatically the following components:

- **Backup Definition (superordinate container)**
- **Backup Agent (the server we’re backing up)**
- *Filesystem Selection (what files we’re backing up)*
- System State Selection (whether we’re backing up the system state)
- Backup Job Default (the default template which we will base this on)
- Backup Definition Name
- **Backup Task**
- **Backup Definition**
- Retention Period (in hours)
- *Schedule*

Items in bold must be created using a cmdlet, items in italics can be specified as a string/int, or as an array/cmdlet for extra flexibility, and normal items are only specified by string or integer.


## Creating the Backup Definition

The cmdlet used to create a new definition is

```
New-BEBackupDefinition
```

But before we do that, we need to specify which server we’re going to be backing up to, which will be done using:

```
Get-BeAgentServer "server11*"
```

You’ll note the asterix at the end. This is to allow for the possibility of a fully qualified server name (i.e. a server name with the domain on the end). Unfortunately Backup Exec doesn’t seem to be consistent in adding servers either with simply the hostname or fully qualified.

So a quick and simple bringing together of these commands would be the following:

```
$agentServer = getBeAgentServer "server11*"

New-BEBackupDefinition -BackupJobDefault BackupToDisk -FileSystemSelection "C:\*" -SystemStateSelection Excluded -name "New Backup Definition" -AgentServer $agentServer
```

As you can probably guess, this will create a backup-to-disk definition called “New Backup Definition” that backs up the C: drive, but not the system state, of a server called “server11”.

**Gotcha:** If you have a slightly myopic naming convention, this will also match “server111”, there are easy ways around this, but I’ll cover that in a later part.

### Multiple Directory Selection & Exclusions

The code above is all well and good if you just want to backup the C: drive, but what if you want to backup, say, the Z: drive as well? Simple! Put them in an array!

```
$selection = @("C:\*", "Z:\*") New-BEBackupDefinition [...] -FileSystemSelection $selection [...]
```

Easy, right? But what if you want to *exclude* a directory? Then you’ll need to use our new friend:

```
New-BEFileSystemSelection
```

For example, if we want to include C: and Z:, but *exclude* C:\Windows, we would do something like the following:

```
# Include the following directories
$selection= @(New-BEFileSystemSelection -path "C:\*" &nbsp;-PathIsDirectory -Recurse)
$selection += New-BEFileSystemSelection -path "Z:\* -PathIsDirectory -Recurse

#Exclude the following directories
$selection += New-BEFileSystemSelection -path "C:\Windows* -PathIsDirectory -Exclude
```

And then pass it to `New-BEBackupDefinition`.


# The Script so Far

So far we haven’t done anything especially complicated, we’ve created a backup definition from defaults, with customised selection and pointed at the client we specified. So far our script looks like this:

```
# Select the correct server
$agentServer = getBeAgentServer "server11*"

# Include the following directories
$selection= @(New-BEFileSystemSelection -path "C:\" -PathIsDirectory -Recurse)
$selection += New-BEFileSystemSelection -path "Z:\* -PathIsDirectory -Recurse

# Exclude the following directories
$selection += New-BEFileSystemSelection -path "C:Windows* -PathIsDirectory -Exclude

# Create the Backup definition based on defaults
New-BEBackupDefinition -BackupJobDefault BackupToDisk -FileSystemSelection $selection -SystemStateSelection Excluded -name "New Backup Definition" -AgentServer $agentServer
```

It’s to important to note that at this point, the definition will contain the *tasks* defined in the default “BackupToDisk” job, namely a Full and an Incremental with fairly short retention periods.

How do we customise the tasks? You’ll see in[ Part 3 Customising and Adding Backup Tasks](http://wp.me/pBnHz-49)!

