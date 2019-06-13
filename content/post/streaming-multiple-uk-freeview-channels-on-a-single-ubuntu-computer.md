+++
author = "toukakoukan"
date = 0001-01-01T00:00:00Z
description = ""
draft = false
slug = "streaming-multiple-uk-freeview-channels-on-a-single-ubuntu-computer"
title = "Streaming multiple UK Freeview channels on a single Ubuntu computer"

+++

I was recently given the challenge of setting up an IP-TV system for a school’s accommodation block, approximately 40 rooms.

Initially I was very skeptical about the whole affair, even working on the assumption that all the kids had laptops (which was a fair one as they’re all rich kids) my last soireÃ© into streaming video was back in 2003 using the then embryonic shoutcast video support at approximately 160kbps.

VLC was the natural choice and had recently released the much touted 1.0.0 version so I decided to have a play around with it on my own computer before comitting to the project and claiming I could do it.

My initial results were… not promising, the video jerked and juttered, the message log filled with errors and in certain modes I couldn’t get the client computer to connect at all.

After much tweaking I decided upon **MMS** as the protocol as any Windows PC with XP or above can simply click on an MMS:// link and open it without issues. I also discovered that the key point to resolving the stuttering and artefacting was to **resize** the thing… re-encoding on the fly at native resolution did not work at all.

**Final settings:  
<span style="font-weight: normal;">Protocol: MMS  
</span><span style="font-weight: normal;">Width: 320</span>  
<span style="font-weight: normal;">Bitrate: 600kbps</span>**

**<span style="font-weight: normal;">With the proof of concept under my arm and a computer with six DVB adapters (I used the [Happauge Nova-T TD-500](http://www.ebuyer.com/product/113946) x 3) I set about figuring out how to do it in Ubuntu 9.04 Jaunty Jackalope.</span>**

**<span style="font-weight: normal;">To cut a long story short, here’s what you need to do:</span>**


# Installing the programs

**<span style="font-weight: normal;">Setup the VLC Installation PPA in *System->Admin->Software Sources *click on the “3rd party” tab and add in the following:  
</span>**

deb http://ppa.launchpad.net/c-korn/vlc/ubuntu jaunty main

**<span style="font-weight: normal;">Then open a command prompt and enter:  
</span>**

sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 7613768D

**<span style="font-weight: normal;"> If that doesn’t work, these instructions are probably out of date, go to the [VLC Website](http://www.videolan.org) for more recent ones.  
 Now install dvb-apps, kaffeine, smb and VLC by going into:  
*System->Admin->Synaptics Package Manager*  
 (smb is **IMPORTANT** as otherwise your windows computers won’t recognise the computer’s hostname)</span>**


# Finding the channels

**<span style="font-weight: normal;">Open Kaffeine *(Applications -> Sound/Video -> Kaffeine)*, press C to open Channels, tell it to scan for new channels.  
 If you can’t do this you’ll have to go into Kaffeine’s DVB settings and tell it which broadcast tower you’re nearest, find that out by putting your postcode in [here](http://www.digitaluk.co.uk/).  
 Once it’s scanned for new channels add them to the list and double click on the first channel you wish to stream.  
 Make a note of the </span>Frequency<span style="font-weight: normal;"> and </span>Service ID<span style="font-weight: normal;">, these are what you need to know to tune the correct channel. Don’t worry if two channel’s frequencies are the same, they’ll be differentiated by the Service ID</span>**


# Starting the VLC streams by command line

<span style="font-weight: normal;">Create a shell script file in your Home directory (right click, new document, give it a .sh extension) I called mine *loader.sh*  
 Open up a command line (*Applications->System Tools*) which should open in your Home directory (*dir* to make sure you can see your shell script file) type ***sudo chmod +x loader.sh***, this changes the permissions on the file to allow you to execute it.  
 Edit the now executable shell script by double clicking on it and selecting ‘View’</span>

<span style="font-weight: normal;">Now here’s the tricky part, you need to create a script that will load up an instance of VLC for each channel you’re streaming (yes, I know that you can run multiple streams from one instance, but I assume that with a multi-core processor it’s more efficient to have seperate processes).</span>

<span style="font-weight: normal;">Mine went along the lines of:</span>

> #BBC 1 on adapter 0 port 1234 cvlc dvb:// --dvb-frequency 706000000 --dvb-adapter 0 --program=4163 --sout '#transcode{vcodec=WMV2,vb=800,scale=1,width=320, acodec=wma2,ab=128,channels=2,samplerate=44100}:std {access=mmsh,mux=asfh,dst=0.0.0.0:1234}' & #BBC 2 on adapter 1 port 1235 cvlc dvb:// --dvb-frequency 706000000 -dvb-adapter 1 --program=4227 --sout '#transcode{vcodec=WMV2,vb=800,scale=1,width=320, acodec=wma2,ab=128,channels=2,samplerate=44100}:std {access=mmsh,mux=asfh,dst=0.0.0.0:1235}'

*It’s important to ensure this is all on a single line (it can be hard to tell as the default editor has word wrap enabled by default).*

<span style="font-weight: normal;">Now, if you only want to broadcast BBC 1 and BBC2 </span><span style="font-weight: normal;">**and** live in Newbury, you can copy and paste that verbatim, but for everybody else I’ll explain what’s going on.  
 The most important bits are:  
*–dvb-frequency* Replace this number with the frequency you got from kaffine, don’t forget to add 000 as VLC takes it in hz and Kaffeine gives it in khz  
*–dvb-program* Replace this with the Service ID Kaffeine gave you  
*–dvb-adapter * Replace this with the adapter number you wish to tune. You can see what adapters are installed by browsing to /dev/dvb  
*:1234* This is the port which your clients will connect to, I simply increment it by one for each channel I’m broadcasting, BBC1 =1234 BBC2 = 1235. If you have the same port twice bad things will happen, so make sure each one is unique! </span>

<span style="font-weight: normal;">The bit in red is the simply the output from the final page of the streaming wizard, so if you want to use a different transcoding setup, simply setup the stream through the VLC GUI and copy everything after the equals sign to replace the red section of the command.</span>

<span style="font-weight: normal;">If you want to add more channels to stream, simply copy the second command (everything from the & to the end of the red quotes) and change the appropriate switches (frequency, program, adapter and port number).  
 The ampersand is important otherwise the script won’t get past executing the first command.</span>


# Starting the script automatically

<span style="font-weight: normal;">As you’ve already made the script executeable earlier, simply go to:</span>

<span style="font-weight: normal;">*system -> preferences -> startup programs*</span>

<span style="font-weight: normal;">*<span style="font-style: normal;">Add a new one with whatever name you like with the path to your shell script eg:</span>*</span>

<span style="font-weight: normal;">*<span style="font-style: normal;">*/home/[username]/loader.sh*</span>*</span>

<span style="font-weight: normal;">*<span>*<span style="font-style: normal;">Replacing loader.sh with the name of your script!</span><span>Â </span>*</span>*</span>Now when your computer starts up, it should automatically start streaming TV!

If you want to start the script automatically, first make sure VLC’s not already running (*System->Admin->Process Manager*) and if it is, kill the processes. Then open a command line, navigate to your shell script, then type:

./loader.sh

Replacing loader.sh with the name of your script!


# Making a user interface

Install Apache and the Apache manager (using the Synaptics Package Manager).  
 Go to the root directory (*/var/www* by default) and delete whatever’s in there and create a new file called index.html  
 Edit this and create links to your streams eg:

<a href="mms://TV:1234">BBC 1</a> <a href="mms://TV:1235">BBC2</a>

<span style="line-height: 18px; white-space: pre-wrap;">Where </span>*TV*<span style="line-height: 18px; white-space: pre-wrap;"> is the hostname of the computer and </span>*1234*<span style="line-height: 18px; white-space: pre-wrap;"> is the port number in question…. I think you can figure out what BBC 1 and 2 are!</span>

<span style="line-height: 18px; white-space: pre-wrap;">After that it’s just a matter of going to the client PC and putting *http://[hostname]* into the browser, eg;</span>

<span style="line-height: 18px; white-space: pre-wrap;">*http://TV*</span>

<span style="line-height: 18px; white-space: pre-wrap;">and then clicking on the link, which will then open up the channel in Windows Media Player!</span>

Obviously with a bit of HTML knowledge you can make it look much prettier.

<span style="line-height: 18px; white-space: pre-wrap;">And there we have it, I hope this has been useful to somebody as it took a little while for me to get right the first time round.</span>

<span style="line-height: 18px; white-space: pre-wrap;">As far as scaling goes I haven’t done a full scale test yet, but a very low spec PC managed to stream to three computers simultaneously without problems so I hope that the quad core beast I setup in the final scenario won’t have any difficulty with 42 users!</span>

If you have any success/issues I’d love to hear from you in the comments.

