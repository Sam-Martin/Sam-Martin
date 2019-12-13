+++
author = "Sam Martin"
date = 0001-01-01T00:00:00Z
description = ""
draft = true
slug = "velocity-conf-amsterdam-2016"
title = "Velocity Conf Amsterdam 2016"
aliases = ['/velocity-conf-amsterdam-2016/']
+++

# Work as imagined and work as done: Mind the gap - Steven Shorrock
[Slides](http://cdn.oreillystatic.com/en/assets/1/event/167/Work%20as%20imagined%20and%20work%20as%20done_%20Mind%20the%20gap%20Presentation.pdf) - [@StevenShorrock](https://twitter.com/StevenShorrock)
![](/images/2016/12/blog---velocity--1.png)  
The core thesis of this talk was that there is a fundamental disconnect between work as you predict it will be done (work as imagined) and work as it is actually executed on the ground floor (work as done).  
While seemingly a truism on the face of it (we all know from our attempts to predict project timelines how hard to predict work as done can be) the lesson Steven wants us to take from this is that we should not try harder to have work-as-done reflect work-as-imagined, but the reverse. Rather than using post-mortems as ways to re-enforce policy, make it clearer and more precise, we should use them to drastically overhaul policy and procedure to reflect work-as-done, even if it's wildly out-of-sync with the vision. Without doing this your documentation become wildly unrealistic aspirations of what *might* be done in ideal circumstances, but are simply whips to beat people with when things go wrong.  
He goes on to reinforce a familiar refrain that objectives, KPIs, league-tables, etc., often have unintended consequences as people find ways to game them to produce favourable results.

**Takeaway:** Don't look to policies to understand your business practices, look at the work that actually occurs. 

# The Web Performance Working Group and You - Yoav Weiss 
[Slides](https://yoavweiss.github.io/webperfwg_keynote_velocity_ams/#1) - [Video](https://www.oreilly.com/ideas/the-web-performance-working-group-and-you) - [@yoavweiss](https://twitter.com/yoavweiss)  
In this talk Yoav takes us through some of the features soon-to-be-available in modern browsers to assist with preloading content such as:
### The `performance` JS API   
https://developer.mozilla.org/en-US/docs/Web/API/Performance/timing
This will allow you to use JavaScript pull out timing data such as Time to First Byte, DOM Content Loaded time, etc, without having to manually measure it using 3rd party libraries.  
 
https://developer.mozilla.org/en-US/docs/Web/API/Performance/mark
https://developer.mozilla.org/en-US/docs/Web/API/Performance/measure
These allow you to add your own entries into the performance log, allowing you to measure things like single-page apps' load times easily.

### Resource Hints
https://w3c.github.io/resource-hints/
These hints allow the browser to preload pages, scripts, etc. that may be required once the user navigates to a subsequent page. They will be loaded after the main content of the page, and hopefully before the users' browser needs them!
`preconnect` prompts the browser to pre-warm an HTTP connection to a domain that may be required subsequently.

``` 
<link href="1.js" rel="preload" as="script">
```
```
<link href="https://3rdparty.com" rel="preconnect">
```

### First Paint
Yoav was also the first to introduce me to a term I would hear a lot in the rest of the conference, the concept of "first paint", and explained the difficulty in defining a 'first meaningful paint'.
First paint is essentially the first time that content is drawn on the screen when a page loads. It doesn't need to be an image, or text, but can be any visual distinction from the white page initially displayed when loading a site.
This may however look simply like a wireframe, or a nav bar, or even just a logo. So Yoav and the WPWG are trying to come up with a programmatic/universal definition of when the 'meaningful' paint has been made, when the page becomes useful to the user.

