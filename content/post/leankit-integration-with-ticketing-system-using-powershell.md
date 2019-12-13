+++
author = "toukakoukan"
categories = ["devops", "kanban", "leankit", "powershell"]
date = 2014-04-20T00:00:00Z
description = ""
draft = false
slug = "leankit-integration-with-ticketing-system-using-powershell"
tags = ["devops", "kanban", "leankit", "powershell"]
title = "LeanKit integration with ticketing system (using PowerShell)"
aliases = ['/leankit-integration-with-ticketing-system-using-powershell/']
+++

I’m no expert on Kanban by any means, but ever since reading [The Phoenix Project](http://www.amazon.co.uk/Phoenix-Project-DevOps-Helping-Business-ebook/dp/B00AZRBLHO/ref=sr_1_1?ie=UTF8&qid=1398001142&sr=8-1&keywords=the+phoenix+project "Amazon: The Phoenix Project"), I’ve been dying to try it out in the workplace.

For me, there are four key things that I think Kanban can help us do that our current tools can’t:

1. Identify queue time & bottlenecks
2. Visualise Work In Process (WIP)
3. Prioritise work (Queued or In Process)
4. Promote mono-tasking by enforcing WIP limits


# History

However, we’ve recently come out the far side of a meta-project to improve the visibility and predicitability of project work by introducing a tool called [Clarizen](http://clarizen.com "Clarizen"), which has resulted in the tool being scrapped.

While I think there are a lot of things that the Clarizen team could do to improve the usability of the product, it could be the easiest to use, most visual, and most efficiency-promoting tool in the world, and it still wouldn’t have gained widespread adoption in my team.

Why? Because it’s Yet Another Input.

We already have:

1. Emails (as much as we try to funnel work through the ticket system, this will always be an input).
2. IM/Lync (as emails)
3. Ticket system
4. Monitoring system (alerts don’t raise tickets currently, though I’m toying with proposing the idea. It’s a matter of how to reduce spam).
5. Personal to-do list (some of the team use Wunderlist, others use Outlook, I’ve got a couple of post-its hanging round in addition).
6. Meetings (hopefully these feed into 3 or 5, or at least 1)

Adding Clarizen on top of that (which, to be truly effective, must be kept up to date as the tasks progress), especially as another location ‘to look’ for work, turned out to be too much. The team tried their best to keep it up to date, but usually the frequency and reaction was “Oh damn, I really should check my Clarizen queue”, me being one of the worst offenders in this regard.


# Rationale

With this in mind, for Kanban to succeed, it needs to reduce the total amount of work without increasing the amount of meta-work/distraction for the team. This means that we need a single place, a canonical source, to look at for deciding what to work on next.

Ideally, this might be Kanban. As I alluded to at the start, our existing ticketing system is poxy awful for assisting us in prioritisation, visualisation and enforcing work in process. So why not use Kanban as canon and the ticket system as reflective? Three reasons:

1. Inertia (as much as we hate our current ticket system, it’s wormed its way solidly into our workflow)
2. Customer facing (customers raise work through our tickets, and we can’t expect them to derive data from our Kanban board as to progress!)
3. Richness of progress &#128;&#147; Kanban lanes are great for a birds’ eye view, but sometimes you just have to scribble down an IP address

So, given that for the above reasons we have to keep the ticket system, (and thereby if we want to run Kanban, do it in tandem), how do we minimise the meta-work of synchronising the state of work between the Kanban board and the ticket system?


# PowerShell!

Why PowerShell? Our team is a Windows-centric operations team; the first half of that description shies us away from Python, Ruby, etc. and the second half from C#, VB.net, etc., as we want something as script-like as possible.

I did originally try bringing the [LeanKit API Client Library for .NET](https://support.leankit.com/entries/28685527-LeanKit-API-Client-Library-for-NET "LeanKit API Client Library for .NET") into PowerShell using* [System.Reflection.Assembly]::LoadFile()*, but ultimately <del>couldn’t get it to authenticate successfully </del>succeeded with @JohnDMathis’ help!

The full post detailing the technical aspects can be [found here](http://samuelmartin.wordpress.com/2014/04/20/using-the-leankit-api-with-powershell/ "Using the LeanKit API with PowerShell"); this one covers the idea behind it, the basic methodology, and the limitations of the current setup from a Kanban perspective.


# Methodology

> Tickets to Cards

Our ticket system is not tailored for our purposes, so the only metadata we can use is:

- Title
- Assignee
- Last Updated
- Ticket number
- Priority

Still, that gives us reasonable flexibility to do as we will with the card once it’s in our Kanban board.

Naturally, we need to turn on the “Card ID” option on our Leankit board so that we can identify which cards correspond with which tickets. It also gives us a nice header across the top showing the ticket number.

Because we have such limited metadata concerning the actual status of the ticket, we’re very limited as to the complexity of the lanes we can have. Our Kanban board is currently just “Queue”, “Assigned” (broken down into team-members), and “Done”.

This doesn’t reflect anywhere near potential of Kanban to achieve our first target at the beginning of this post (*Identify queue time & bottlenecks*), but it’s a start for the other objectives. Greater granularity to unleash the full potential of Kanban will have to wait until *Phase 2*.

Temporarily, we’ve assigned an arbitrary amount of time to add to the “last updated” value from the ticket and set that as the “due date” of the card, to give us an indication of the ‘freshness’ of the task. There may be a better way of visualising this, but we’re still experimenting.

The workflow of the script goes as follows:

1. Get tickets and cards
2. Identify new tickets which don’t have cards
3. Create cards for new tickets
4. Refresh list of cards
5. Loop through cards, identifying those which need updating
6. Submit updated cards

Simple, but crucially, in step #5, only the lane and priority is updated. This means we can rename cards, change their type, expedite them, etc., and the changes won’t get overwritten the next time the script runs.

This allows the people who need to understand the overall view to manipulate the WIP in a way that’s meaningful to them, without either burdening the team members with updating metadata which is not useful to *them*, nor requiring the people interested in the enriched data to go into individual tickets and update them themselves.


# Downsides

Without modification, there are a few downsides to this idea. We can’t achieve any of the following:

- Bottleneck identification (source data not rich enough to auto-populate more than 3 lanes)
- Wait-time identification (source data not rich enough)
- Rich workflow representation (source data not rich enough)

All of which are problems stemming from the fact that we’re updating the lane from the ticket. If we were to simply use the broker script as a way of inputting the cards initially, we would be able to increase the lane complexity and the above problems would go away.

However, they would be replaced with the synchronicity<span style="color:#252525;"> </span>problem, something we’re not ready to tackle yet.


# Phase 2

Ultimately, we want to be able to support a more workflow-reflective Kanban board based on the ticket metadata. This could be done either by having meaningful ticket categorisation and statuses, or by achieving full buy-in from every member of the team as to the importance and efficacy of Kanban. But I suspect the latter option would, as observed with Clarizen, come at the price of one system or another.

Hopefully the former option will be realised with future updates to (or replacement of) our ticket system. But if anyone has any other ideas how to achieve lane complexity without increasing the work of the team in maintaing two systems, I’m all ears!

