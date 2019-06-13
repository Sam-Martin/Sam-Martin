+++
author = "Sam Martin"
date = 2016-05-26T00:00:00Z
description = ""
draft = false
image = "/images/2016/05/logo.png"
slug = "winops-conference-london-2"
title = "WinOps Conference London #2"

+++

# Intro
<img class="js-upload-target" src="/images/2016/05/logo-1.png" style="width:6em;float:left;">
WinOps is a conference in London aimed at addressing the fledgling audience of Windows people trying to DevOps. It's also a monthly UserGroup which I attend regularly, so I was excited to see how the content translated from UG to conference.

I don't have slide decks for all of the talks (though I've added them where available), if you have one that I'm missing, please link to it in the comments!

# Jeffrey  Snover - The DevOpsification of Windows Server 
After hearing and seeing Snover on so many podcasts, Virtual Academy videos, and conference recordings, it was great to finally see the inventor of PowerShell talk in person. Sporting [Vibrams](https://www.vibrams.co.uk/collections/mens-vibram-fivefingers), Snover took the stage to run the audience through the historical 'phases' of Windows Server to frame what he sees as the next stage of evolution, Windows Server 2016.
I love listening to Snover talk as he's one of the few people at Microsoft who you'll hear speak in an opinionated fashion. You *should* do this, you *must* do that, you *shouldn't* do that. It's very refreshing, and I've yet to disagree with his positions, his insight and vision is pivotal in Microsoft's recent successes.

In terms of content, if you've been following the blog/podcast/conference scene you'll be familiar with the overview provided in this talk, covering: 

* TDD with Pester
* PowerShell 5 
* Package management
* Just Enough Administration (JEA)
* Nano Server
* Windows Containers
* Server And Desktop = SAD Server

If you weren't familiar with a topic, it probably won't have given you a huge insight into what the topic *meant*, but the overview served as an excellent scene setter for the subsequent talks which were aimed to do just that.
# Iris Classon - To the Cloud!
Iris gave us a summary of her company's 'journey to the cloud' with a previously on-premise product. Moving to a multi-tenant PaaS oriented CD-enabled architecture, the journey was a well explained overview of a typical green-field (as opposed to lift-and-shift) project to bring your product into the cloud. The presentation style was quite exceptional, with all the points being live-drawn by Iris, and was consequently very engaging. 

There were lessons to be learned from the story, but it was largely left to the listener to draw them for themselves. My take away was that you can succeed in a large-scope cloud project with limited resources if you make intelligent decisions about what components to make time savings on (e.g. by postponing the loose-coupling of a small-scale component until it's absolutely necessary).

## Ed Wilson - Config Management with Azure Automation
Ed took us through the way DSC Pull servers and System Center Operations Manager (SCOM) worked together in Azure to provide a holistic story of configuration management without using Chef or Puppet. Personally my take has always been that if you're not already using SCOM (and are not Enterprise IT focussed) then you should explore the Open Source tools out there first. I'm open to being shown the error of my ways on this topic, but I didn't get the opportunity in this session as I had to leave the auditorium to censor a coughing fit.

# Michael Greene - Microsoft Release Pipelines
I'd heard of Michael from a [RunAsRadio episode with Steve Murawski](http://runasradio.com/Shows/Show/469) recently where Steve spoke about the whitepaper that he and Michael had written about [The Release Pipeline Model](http://aka.ms/thereleasepipelinemodel).

Michael's talk was definitely the highlight of the conference for me. Although aimed at providing an introduction to release pipelines to the audience (commit, build, rave, repeat), Michael's humorous presentation style, combined with some extremely pithy insights into relatable scenarios kept me smiling and engaged throughout.  

His comments regarding typical ITIL change processes were very relatable to me. Change Control so often loses sight of what value it adds and ends up focussing solely on building an audit trail for the benefit of certification. He reminded us all (and perhaps enlighted more than a few!) that we can achieve the main goals of CC through common DevOps processes.

1. Visibility (Infrastructure as Code/ChatOps)
2. Record of change for troubleshooting (Source control)
3. Rollback processes (Standardised build and deploy pipelines)

I'd love to see a talk where Michael drills into one or two elements of his pipeline model in more depth as I think he has a huge amount to say on the topic.
Oh yeah and don't forget to go and read his and Steve's whitepaper: https://msdn.microsoft.com/en-us/powershell/dsc/whitepapers#the-release-pipeline-model
You can find the slides for his talk [on Michael's Github page](https://github.com/mgreenegit/slides/raw/master/TRPM/TheReleasePipelineModel_WinOps.pptx). One particular side right towards the end where Michael outlines a "getting started" guide to DevOps tools on Windows (featuring Pester, PSake, PoshSpec, and PSDeploy/Lability) is particularly worth checking out.
# Richard Siddaway - DevOps with Nano Servers/Containers
Richard provided a fast-and-furious run through of building a Windows container image in 2016, that really has to be seen to be understood. I will add the video to this blog when it's made available.
# Panel - DevOps Culture in a Windows World

* Amy Harms, [WorldRemit](https://www.worldremit.com/)
* Hannah Foxwell, [Pendrica](https://pendrica.com/)
* Steve Wade, [Steven Wade Consulting](http://www.stevenwade.co.uk/)
* James Smith, CEO, [DevOpsGuys](https://www.devopsguys.com/)

One of the things I love about the rise of the Windows DevOps movement is that it's resulted in a lot more talks relating to corporate culture and DevOps, which are much rarer in the wider DevOps space as the cultural challenges tend to be of a less enterprisey nature.

Of the questions and answers in the session, these stood out for me in particular:

> ... Because we manage *projects* not *products*  .

This was in answer to a question concerning how to deal with technical debt, and specifically, why do we end up with so much 'legacy' code/infrastructure. Quite apart from the initial comment from one of the panel that in her company legacy is a banned word (as if it's running in production, it's just *production*), the concept of running projects not products is one that rings very true for me and is one I'd cite as a lesson from the last couple of years. Project teams work fine and can make a lot of sense when they're multi-skilled teams drawn from existing product oriented groups as a temporary team. But ultimately you're only going to get the attention, care, and resultant evolution of your product when you have full stack teams oriented around that product or a few like it

> What's your #1 tip for implementing DevOps

1. Involve everyone
2. Find a painful (but achievable) problem and concentrate on fixing it with DevOps
3. Don't try to change culture directly as you'll just get brickwalled

# Matteo Emili - Development & QA Dillemas in DevOps
Matteo took us through a series of deployments in Azure using ARM and particularly introduced me to two new ideas:

1.  Feature Flags (i.e. the ability to turn a feature on or off in your product just by toggling a value)
2. Silent Releases (to break up big changes/feature releases into smaller chunks of more manageable change without necessarily providing new functionality in each release). 

I hadn't actually heard of either of these two concepts previously, but looking at them they seem like such simple and obvious solutions to a very thorny problem of delineating large releases. (This hindsight is of course the hallmark of all the very best ideas!)

#Panel - What does the Future Hold?
This was the closing session of the day, and ended the conference on a lovely humorous note with the question "What was the funniest thing you learned during an outage?". I unfortunately wasn't taking notes for this session, but if anyone with a better memory than I can fill in the answers to this questions, leave them in the comments!

# Closing Thoughts
WinOps as a conference was squarely aimed at people who are new to DevOps, providing introductory material and high-level overviews. While this makes it somewhat less valuable to me I still got a lot out of it in terms of individual tool recommendations, introductions to new and engaging speakers, and of course the hallway track. 
I think it's also absolutely the right fit for their audience. DevOps is a very new concept in the Windows world, and a lot of the people attending are looking for exactly that introduction. Michael Green's talk in particular is *exactly* the sort of talk I was craving for but never got when I was trying to get my head around DevOps originally. Speaking to some of the attendees who'd been to the previous (and first) WinOps conference, I got the suspicion that this conference was much more introductory than the first, and my guess would be that that change is a result of the feedback of the attendees.

I wouldn't be surprised if they got the exact opposite feedback from the attendees this time round, and the suggestion of a colleague of mine to have two tracks (one at 100 level and another at 200-400 level) seems like the best way to please everyone and grow the conference to boot.

All told I really enjoyed WinOps, even if I wasn't the target audience!

# Links
[Photos from WinOps #2](https://www.facebook.com/WinOpsConf/photos/?tab=album&album_id=1724700647804027)  
[WinOps.org](http://winops.org/)

