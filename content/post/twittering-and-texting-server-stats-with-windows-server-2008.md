+++
author = "toukakoukan"
date = 2011-04-04T00:00:00Z
description = ""
draft = false
slug = "twittering-and-texting-server-stats-with-windows-server-2008"
title = "Twittering (and texting) Server Stats With Windows Server 2008"
aliases = ['/twittering-and-texting-server-stats-with-windows-server-2008/']
+++

Even though the RAM usage issue is *hopefully* resolved, I’m not taking any chances. So I decided to set it so that the server will direct message me on Twitter when the RAM usage reaches a certain threshold.


# Step 1: Perfmon

To start the performance monitor, simply type ‘perfmon’ in the run command or open it up from ‘administrative tools’

You’ll need to create a user defined **Data Collector Set**

1. Choose to create manually, hit next,
2. Select ‘Performance Counter Alert’, next,
3. Add the performance counter you’re interested in  
 (I chose Memory -> Available Megabytes) and choose your alert conditions (I chose below 512mb).
4. Choose the user to run it as (system should be fine)

First edit the new Data Collector set and setup the schedule, else it’ll stop running the next time you restart your server!

Now edit the newly created Data Collector inside the Data Collector Set and configure the following

1. Sample Interval, 15 seconds will be *way* too often to to update twitter, choose a minute for the time being, you can change it to an hour or whatever you like once we’ve got it up and running
2. Alert Task, this can be whatever you like, but **remember** what it is, as we’ll be creating a scheduled task with the same name in a moment
3. Alert Task Argument, for the sake of simplicity set it as {value}


# Step 2: Scheduled Tasks

Open up the Task Scheduler MMC from Administrative tools, right click on the  Task Scheduler (local) and hit ‘create task’ **(not ‘create basic task, that’s no use!)**

1. Give it a name, the **has to be** same name you gave the Alert Task in the performance monitor
2. Check to ‘run whether user is logged in or not’
3. Actions -> New Action
4. Set the action as send an email to your [twittermail.com](http://www.twittermail.com) address.
5. Set the text to whatever you like, the important thing to know is that the argument you passed to the task earlier will pop up as `$(Arg0)`, so my message read:  
 `d <twittername> The server has  $(Arg0)Mbytes of physical memory free.`
6. Set the SMTP server to be whatever you’re going to use.  
 You can set up IIS and it’s SMTP server if you have no other, just put in localhost or 127.0.0.1 if you’re doing this. I did this and stumbled over the default behaviour of the “Simple Mail Transfer Protocol” service not starting up automatically, so make sure you go to services.msc and change that behaviour if you do this!

Make sure you start the Data Collector Set you just created in perfmon, and bear in mind that when you make changes (such as the sample interval) you’ll need to stop and start it again.

**That’s it!**

If you need the nudge, you achieve the texting or SMSing by configuring twitter to send you an SMS every time you get a direct message.

Let me know how you got on in the comments!

