+++
author = "toukakoukan"
categories = ["cfgmgmtcamp", "devops", "hashicorp", "juju", "vault"]
date = 2016-02-15T00:00:00Z
description = ""
draft = false
image = "/images/2016/02/2015-02-07-Juju.png"
slug = "configuration-management-camp-2016-impressions"
tags = ["cfgmgmtcamp", "devops", "hashicorp", "juju", "vault"]
title = "Configuration Management Camp 2016 Impressions"

+++

At the beginning of February I was lucky enough to attend [Configuration Management Camp](http://cfgmgmtcamp.eu) for the 3rd year running. The first year I attended (2013) it was my first introduction to the concept of configuration management, and god damn it was overwhelming. So many new concepts, new tools, new ideas, I left having been enlightened to a whole new world of automation.

This year, having *massively* improved my knowledge of modern tools and practices around automation and infrastructure, and now having worked with Chef for a couple of years, I came to cfgmgmtcamp with the goal of learning more about Docker in production. But actually the amount I learnt about Docker was relatively slim (other than that every Docker talk starts with a picture of a toppled container ship, and that networking at scale with Docker seems really *really* hard).

Three talks stood out for me.


## [Secrets, Certificates, and Identity with Vault](http://lanyrd.com/2016/cfgmgmtcamp/sdxwrr/)

<iframe allowfullscreen="" frameborder="0" height="585" src="https://www.youtube.com/embed/sp_Re8Mx9xk?start=8859&feature=oembed" width="1040"></iframe>

I’d heard of Vault before, and tentatively had it on the backlog for PoC and implementation, but didn’t really know a great deal about it other than that it was an open source secret management solution intended primarily for programmatic access.

What I learned *additionally* in [@mitchellh](https://twitter.com/mitchellh)‘s talk was that:

1. It allows you to manage other secret management systems (e.g. AWS IAM) by mounting secret [backends](https://www.vaultproject.io/intro/getting-started/secret-backends.html).
2. Further, it allows you to [dynamically manage the secrets](https://www.vaultproject.io/intro/getting-started/dynamic-secrets.html) in these backends to e.g. facilitate access key rotation.
3. It has a rather clever [split key](https://www.vaultproject.io/docs/concepts/seal.html) methodology to alleviate the [turtles-all-the-way-down](https://en.wikipedia.org/wiki/Turtles_all_the_way_down) problem of having a master key that allows you to unlock the vault.
4. Promotes an authentication methodology for your application called [app-id](https://www.vaultproject.io/docs/auth/app-id.html), which uses a common auth secret (which you might store in your configuration management system) in combination with an node specific secret (e.g. a salted derivation of the MAC address) to create a unique ID that allows the machine to auth against Vault.


## [Orchestration? You Don’t Need Orchestration. What You Want Is Choreography](http://lanyrd.com/2016/cfgmgmtcamp/sdxytm/)

<iframe allowfullscreen="" frameborder="0" height="356" marginheight="0" marginwidth="0" scrolling="no" src="https://www.slideshare.net/slideshow/embed_code/key/KyCVYxsDGd60GL" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" width="427"></iframe>

<div style="margin-bottom:5px">**[Choreography? You Don't Need Choreography. What You Want is Orchestration](https://www.slideshare.net/JulianDunn/choreography-you-dont-need-choreography-what-you-want-is-orchestration "Choreography? You Don't Need Choreography. What You Want is Orchestration")** from **[Julian Dunn](http://www.slideshare.net/JulianDunn)**</div>I will probably butcher the explanation of the concept that [@julian_dunn](https://twitter.com/julian_dunn)  ([juliandunn.net](http://www.juliandunn.net/))was conveying in this talk, but my take away was as follows.

Orchestration systems as a centralised job execution systems work by trying to enforce ‘obligations’ on nodes to  try and bring about a desired end-state. This puts the onus of responsibility for workflow exception handling on the orchestrator, rather than the node. When the orchestrator is trying to handle a large number of workflows with heterogeneous components, this centralised execution enforcement model becomes incredibly fragile. The amount of thought, detail, and ultimately code that needs to be put into the orchestrator to make it reliable becomes totally untenable.

Far better, to extract a [promise](https://en.wikipedia.org/wiki/Promise_theory) from the node that *it* will ensure the successful execution of your desired task, thereby making the node aware of the workflow state it’s currently in. This way, you’re distributing your workflow knowledge throughout your nodes, allowing them to do much more with the state information that they now have.

E.g. you might have a DB server that you’re telling to  upgrade a schema. This server will then publish state information to the webserver that depends on it, telling it “hey, I’m in maintenance mode right now, why don’t you run do your db-in-maintenance-mode thing”. The webserver in turn will then do something intelligent like swap to a holding page that says “maintenance in progress”, rather than throwing a 500 and causing itself (and all the other nodes) to be marked as unhealthy in the LB.

While you could do this with orchestration in theory, it relies on a single locus of control pushing all this intelligence, re-inventing the wheel for each combination of components to ensure that you’re making the best use possible of your orchestration system. Whereas if you push that logic into your nodes/apps/cfgmgmt system then you’re able to define a series of interfaces and behaviours that are applicable for all possible combinations of that component.


## [The Magic of Modelling](http://lanyrd.com/2016/cfgmgmtcamp/sdxwrq/) (and other JuJu track talks)

<iframe allowfullscreen="" frameborder="0" height="585" src="https://www.youtube.com/embed/sp_Re8Mx9xk?start=4096&feature=oembed" width="1040"></iframe>

This talk by [@sabdfl](https://twitter.com/sabdfl), ([Mark ShuttleWorth](https://en.wikipedia.org/wiki/Mark_Shuttleworth)) made me realise I’d been confusing [JuJu](https://jujucharms.com/) and [Mesos](http://mesos.apache.org/) since [Fosdem](https://fosdem.org/) last year.

As far as I understand, [JuJu](https://jujucharms.com/) is the personification of the principal Julian Dunn was talking about in his talk above. It’s an open-source ‘modelling’ tool, that allows you to create ‘charms’, which are in essence wrappers around your favourite config management tool (Chef, Puppet, CFEngine, Bash, it doesn’t care) that define three things:

1. Events (e.g. new database server available)
2. Interfaces (e.g. http, mysql-protocol)
3. Layers (e.g. apache, mysql)

Once these three components are defined and wrapped into a Charm, you can start that charm and start building relationships.

[![2015-02-07 - Juju](/images/2016/02/2015-02-07-Juju.png)](/images/2016/02/2015-02-07-Juju.png)

You might say to yourself “I want an Apache webserver with WordPress, backed by MySQL, fronted by HAProxy”, and you’ll be able to add charms from the store, and literally drag and drop relationships between these charms to build a working web application.

When you create a relationship, e.g. between Apache with WordPress installed, and MySQL, it will trigger an **event** in each system. In our case, this will cause the WordPress node to execute its setup db script against the MySQL **interface**.

Using this event driven system where each charm knows its own capabilities and not only how to instantiate them, but also handle failure, seems **incredibly** powerful. It almost sounds too good to be true, and I’d like to see how it works in practice, but the principles seem sound.

The common trickier operational edge cases like handling application upgrades, HA failover,  data restoration, customisations, disaster recovery, etc. seem like quite a lot of work to embed into the model using events, especially as it would need to be done upfront, but it seems like a very very interesting product to watch.

