+++
author = "toukakoukan"
date = 2011-08-29T00:00:00Z
description = ""
draft = false
slug = "gmail-advanced-upload-wont-work-in-chrome-with-non-administrative-users"
title = "Gmail advanced upload won't work in Chrome with non administrative users."

+++

The other day some of my users complained of a very strange issue when using Gmail at work. When composing a new email, they could not upload attachments using the “Advanced Upload” function. When they clicked “Add File” to upload a new document, nothing happened, no error, nothing, it just sat there and refused to work.

Naturally it worked *fine* when I tried it. It seemed to happen only when the user did not have administrative access on their computer. Odd, I thought.  
 A quick search of the web [revealed](http://www.google.com/support/forum/p/gmail/thread?tid=3396cd44832f693d&hl=en) [quite ](http://www.google.com/support/forum/p/gmail/thread?tid=7f77c144096a5903&hl=en)a [few](http://www.google.com/support/forum/p/gmail/thread?tid=00e43f845b2045d6&hl=en) [people](http://www.google.com/support/forum/p/Chrome/thread?tid=579d5b2939912968&hl=en) [with](http://www.google.com/support/forum/p/Chrome/thread?tid=7edb1b3753e0be40&hl=en) [the](http://www.google.com/support/forum/p/gmail/thread?tid=55345133bb303388&hl=en) [same](http://www.google.com/support/forum/p/Chrome/thread?tid=46ebd2f718483530&hl=en) [problem](http://www.google.com/support/forum/p/gmail/thread?tid=21f7c5a346d7c348&hl=en). But no real solutions other than “Switch to basic upload”, which is hardly a solution. A bit disappointing when a major feature of a Google product doesn’t work with Google’s flagship product!

I did however happen upon a solution myself. It turns out that it’s *somehow* related to the inbuilt flash plugin that comes with Chrome.  
 If you disable that, then install Flash, presto changeo, it works!

You can disable the plugin by typing

> chrome://plugins

into the address bar and disabling it from there, but for administrators it’s probably easiest to use the following command line switch:

> -disable-internal-flash

[![](/images/2011/10/chrome-shortcut-properties.png "chrome shortcut properties")](/images/2011/10/chrome-shortcut-properties.png)

There’s probably a better solution, something that allows the inbuilt Flash plugin to work, but this was an improvement over the solutions posted on Google’s user-based tech support board!

