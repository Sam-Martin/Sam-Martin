+++
author = "toukakoukan"
date = 0001-01-01T00:00:00Z
description = ""
draft = false
slug = "new-scientist-article-limit"
title = "New Scientist article limit"

+++

The New Scientist website has recently started imposing an article limit for non-paying users.

![New Scientist Ad Block](http://imgur.com/H0uXA.jpg "New Scientist Ad Block")

As anyone with any familiarity with JS and jQuery (which, the New Scientist webmasters kindly already supplied) would do, I coded a little bookmarklet to get rid of this annoyance.

> javascript:$('#haasOverlay').hide();javascript:$('#fadeBackground').hide();

To use it, just create a new bookmark in your browser and paste the text above as the URL.  
 When presented with the article limit block, simply open the bookmark and the block will vanish!

