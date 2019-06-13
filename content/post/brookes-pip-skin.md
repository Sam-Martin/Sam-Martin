+++
author = "toukakoukan"
date = 0001-01-01T00:00:00Z
description = ""
draft = false
slug = "brookes-pip-skin"
title = "Brookes PIP skin"

+++

I got bored of Â how the Brookes Personal Information Portal looked.

![How PIP Looked](http://imgur.com/0jUGnl.jpg "How PIP looked")

So I decided to code a little plugin for Google Chrome to make it look a bit prettier.

![How PIP looks now](http://imgur.com/wvDh5l.jpg "How PIP looks now")

Took about 6 hours as the page was so *old* that it was done with no CSS or XHTML whatsoever; tables and in-line formatting all the way. Still, with the help of jQuery’s traversing functions I was able to overhaul it quite easily.  
 I even went so far as to extract the URLs of the navbar (along with their titles) and completely replace the <table> setup with an alphabetised <ul> setup (before, the order changed from page to page, making navigation a case of hunting through the nav-bar for the link you wanted).

If you’re a Brookes student, you can [get it here.](https://chrome.google.com/extensions/detail/mkpekmapcokdappehbpdnpngaffmglei "Brookes PIP Skin - Chrome Plugin")

