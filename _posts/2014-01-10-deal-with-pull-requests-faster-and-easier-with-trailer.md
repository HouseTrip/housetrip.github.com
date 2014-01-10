---
layout: post
title: Deal with pull requests faster and easier with Trailer
published: true
author: Paul Tsochantaris
author_role: Developer
author_url: http://github.com/ptsochantaris
summary: At HouseTrip we have an extensive code review process, nothing reaches production without having been eyeballed and discussed previously.  Each feature or change has its own branch, and gets merged when ready for deployment. So we decided to make Trailer, a simple menu bar app that provides the one feature we felt was lacking from this workflow.
---

### 1cm of menubar is a lot to ask

<img src="/images/2014-01-10/menubar.png" alt="Trailer App in the Menu Bar" style="border:none;box-shadow: none;margin:0;"/>

It takes a lot of nerve to ask a user to give up a centimetre of their menubar for an app that you have made.  It has to be worth it and provide a lot of value in return for occupying that always-visible premium real estate on a screen.

Surely then, a giant app suite that provides an MS-office-like bundle of kitchen-sink feature design applied to Github workflows, with a server-client architecture, an extendible API, plugins, and a flight simulator easter egg in the about box is just the ticket.

Or perhaps, a simple menu that provides the one feature we felt was sorely lacking from our workflow.  Sometimes the feature is the app, and that is good.  So we made Trailer!

<img src="/images/2014-01-10/screenshot.png" alt="Trailer App Screen Shot" style="border:none;box-shadow: none;margin:0;"/>

### The problem

At HouseTrip we have an extensive code review process, nothing reaches production without having been eyeballed and discussed previously.  Each feature or change has its own branch, and gets merged when ready for deployment.  This means we go through a lot of Pull Requests, concurrently, and the faster PRs can be reviewed and commented on, the better.  Add to this that a PR may go through many iterations of review, comments, and resubmission.  For a few developers with a few PRs Github notifications work perfectly well.  When we scale this up, with disparate teams working on different things and on a tight schedule, Github notifications can and do get tuned out.  We needed an instant birds-eye view of the state of our PRs and a motivation to go through them faster.

### The solution(s)

> **Reducing the opportunity cost for starting the review process:**  A developer has a few minutes to spare.  They need to be able to reach out and "grab" a PR.  No opening browsers, navigating pages, or keeping a constant eye out on Skype.  On the flip side, freeing developers from having to advertise their PR to potential reviewers provides valuable time for doing more productive work.  It didn't need deep thought; almost everyone wished aloud that there was a simple menu on their menubar that would list the more recent PRs and take them to their page.  Sorting by age, creation time or last activity time is very important, since PMs for instance want to see if there is activity on a PR, while developers may want to see the oldest PRs on top.  Realtime filtering helped track down specific PRs on the list.

> **Motivate devs to deal with PRs more often:** The answer to this was very simple and straightforward.  Display the total number of open PRs in the menubar.  A slow, leisurely nagging that you don't really mind, except that when that number goes down.  It feels cool in that way only us people who write code can appreciate :)  What is interesting is it turns out as positive reinforcement instead of irritation, which was an initially big concern.

> **Reduce discussion lag:** The overwhelming requirement here was to enable faster responses to comments and vice versa.  Two things were needed.  One was integration into the OSX notification centre.  A lot of apps do this as a bonus, but notifications would be a core part of Trailer, and they needed to do the right thing when selected by the user.  The second thing was to give comments a front-stage presence in the app.  Comments are loud and turn things red.  PRs get badged with new comments.  The PR count on the menubar turns red (a slightly more muted red to be sure, but same difference) This also meant creating a "mark everything as read" option of course.  Comments are only shown on a user's PRs (or PRs a user has commented on) by default, but there is an option to turn it on for everything - something that is more useful to managers, for instance.

> **Reduce time-to-deploy:** Closed PRs are checked for merged status.  If they are merged, the user is notified.  These PRs are not removed from the list, but moved into a "recently merged" section, and have to be removed manually.  This works as both a deployment-ready section and a rough visual guide on the rate of merges.

> **Avoid adding to the chaos:** PRs can grow to huge numbers.  Installing and setting up apps which consume APIs can be tricky.  Well, Trailer doesn't need any complicated installation, it's a simple app bundle, you can run it from anywhere.  To configure it, you just need to paste an existing or new Github API access key, for which we provide a link inside the preferences window, and pick the repositories you want to monitor.  PRs are ordered by relevance in sections: My PRs, Participated Prs (i.e. stuff I've commented on), Recently Merged PRs, and All PRs.  Then we have sorting.  And if you still can't find what you're looking for in the case of hundreds of PRs, then there is filtering.

> **Be nice:** Trailer keeps you notified on your Github API usage limits, warning you if it gets too high.  Its memory usage and network usage is the bare minimum, with ongoing work being done to reduce this even further.  All processes run at the lowest possible thread priority.  You can further reduce this by increasing the refresh period in preferences.  Left alone, Trailer does no background processing whatsoever.

<img src="/images/2014-01-10/prefs.png" alt="Trailer App Screen Shot" style="border:none;box-shadow: none;margin:0;"/>


### What's next

We're thrilled to open source Trailer under the MIT license, and are looking forward to the community's involvement!  

Things that we are already working on are iOS support in the 1.0.2 branch, plus improved stability and performance.

We hope it's as useful to anyone else using Github and pull requests as it is for our workflow, enjoy, and remember to watch the Trailer :-P