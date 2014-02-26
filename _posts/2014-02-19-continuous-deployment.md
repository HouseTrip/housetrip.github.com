---
layout: post
title: Continuous Deployment
published: true
author: Stefan Dorunga
author_role: Developer
author_url: http://github.com/sdorunga1
author_avatar: http://www.gravatar.com/avatar/3368bed08c1a9a93bb9612a374e7714c.png
summary: |
  Do you trust your software and processes? Prove it and do continuous deployment
---
In light of our current move towards a service oriented architecture, we have 
decided to adopt continuous deployment as part of our process. For quite
some time here at HouseTrip we have had Continuous Integration and Continuous
Delivery, so it came as a very natural and straightforward extension of our current
practices.

Before diving deeper into the topic and considering that there are so many 
"continuouses" in this article, I'd like to give a brief overview of what Continuous
Deployment is. The simplest way to explain it is, funnily enough, in relation to
Continuous Delivery.

According to Martin Fowler, you're properly doing Continuous Delivery if:
> - Your software is deployable throughout its lifecycle
> - Your team prioritizes keeping the software deployable over working on new features
> - Anybody can get fast, automated feedback on the production readiness of their systems any time somebody makes a change to them
> - You can perform push-button deployments of any version of the software to any environment on demand
 
The key distinction comes from that last step in the list and is essentially just
a 1UPed version of it. In Continuous Deployment the final step is done automatically
each and every time a change is merged to your production branch and the Continuous
Integration process gives the green signal.

This is very well illustrated by the good folks at PuppetLabs in the following diagram
<img src="/images/2014-01-25/continuous_delivery_continuous_deployment.jpg" alt="Continuous Delivery vs. Continuous Deployment" style="border:none;box-shadow: none; margin:auto;"/>

##Implementation

Because of our adoption of the other two continuous processes, implementation has
been quite easy to do. As we've migrated all our services to Heroku and from 
our custom CI server onto TravisCI, all this took was the following 4 lines of code:

```
deploy:
  provider: heroku
  app: your-app-name
  api_key: "YOUR API TOKEN"

```

Understandably this may be too much effort and the devs at Travis feel your pain,
so they  allow you to use their `travis` tool (available as a gem) to run 
`travis setup heroku` in your project directory which has the added benefit of 
also encrypting your Heroku API key. The [official documentation](http://docs.travis-ci.com/user/deployment/heroku/)
allows for much richer functionality, so if your needs are a little more 'custom',
then do dig in.

##Caveats

Easy as it's been, this switch does cause a bit of confusion. When do we deploy?
Who is deploying? Who is announcing the deploy? Answer to all of these questions:
'Not me!'

It is quite a weird feeling allowing 'the machines' to take over the deployment
process but you do eventually get used to it quite quickly and it becomes quite
freeing to know that, once you've done your job addressing comments on your PRs,
everything will be taken care of for you and your shiny new feature will be pushed
to production.

The two main concerns that arose from this system were that it hides deploy events
and limits you in your ability to decide when to issue a deploy.

In all truth, and at the very least for services, deployments should not be a 'big
bang' event as much as a regular and transparent part of your process and should 
therefore not even require notification. Obviously people do like to be kept in
the loop, and find being notified as a useful feature. To address
that we have just used one of the many integrations available with Travis to 
notify us on a new deploy. 

One exception to this rule are, typically, large database
migrations. This is where you have to be a little bit more clever about your solution.
Normally you would manually handle this process, probably deploying at night (or
whatever your low traffic period is) and take the site off for an arbitrary length
of time. One good way to do it is to take a more incremental approach: write your
code changes in a way that they support both database schemas, deploy(i.e. merge
your code), run migration live, issue a PR to remove the fallback code. This
migration may still lock your tables for longer than you can afford, in which case
you could try dumping all your data into a different table, migrating that and 
switching between the two at the end. Obviously this does have its challenges 
but we have successfully done it in the past.


As for the manual deployments, they are always still an option, and they can happen
outside of the CD process, but they should be minimized with the ultimate goal of
complete automation depending on how well you manage to automate your infrastructure.

Overall, this system provides for a lot more discipline as you are actually 
guaranteed that once your work's been merged it will instantly get deployed. That
makes you think twice about whether you are actually done and ready to issue that
pull-request.
