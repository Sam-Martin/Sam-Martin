+++
author = "toukakoukan"
date = 2011-03-28T00:00:00Z
description = ""
draft = false
slug = "windows-server-2008-cache-management"
title = "Windows Server 2008 Cache Management"

+++

# Or, “How explorer.exe used all my server’s RAM and caused it to die”

One of my clients was experiencing slowdown of their AD server due to explorer.exe consuming 100% of the RAM.

I say slowdown, with that sort of RAM usage the thing ground to a halt and had to either be restarted or have explorer.exe killed (if we were lucky enough to be able to get to the task manager).

After much Googling (and the deadend that was the ‘Disable Search Files Option’ fix) I eventually found this:

**KB 976618: [You experience performance issues in applications and services when the system file cache consumes most of the physical RAM](http://support.microsoft.com/kb/976618)**

through [this technet thread](http://social.technet.microsoft.com/Forums/en-US/windowsserver2008r2general/thread/74c2c9ca-f8c1-4c37-bc8c-cd074ce0c6cd "2008 R2 Memory Caching").

The KB article basically boils down to “Windows will use all your ram, then fall over” and their solution is “Install this series of .reg files and install a service we should have included in the first place”.

Thanks for that MS, you might have put it in an automatic update…

There’s a [blog post](http://blogs.msdn.com/b/ntdebugging/archive/2007/11/27/too-much-cache.aspx) about this issue (specifically mentioning its likely prevalence in x64 systems) from *2007*!

