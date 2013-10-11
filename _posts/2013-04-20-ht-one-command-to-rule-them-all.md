---
layout: post
title: HT - one command to rule them all
published: true
author: Matt
author_role: Developer
author_url: https://plus.google.com/107981766773306423214?rel=author
author_avatar: http://www.gravatar.com/avatar/9e0e76de0fe26c1326da1a232d4dd2f2?s=36
---
[DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself) doesn't just apply as a best practice for software development.  At HouseTrip, we're always looking for better ways to automate things.  Everything we do repeatedly, should be a candidate for optimisation.  From simply connecting to servers, to pulling down stats on our production job queues.

As a team we have amassed a collection of shortcuts, scripts, and rake tasks to make our lives just a little bit easier.  In an effort share amongst everyone, I have decided to formalise them all in a new **ht** command.

## 37signals Sub

The **ht** command uses [sub](https://github.com/37signals/sub), a command line framework from [37signals](http://37signals.com/svn/posts/3264-automating-with-convention-introducing-sub).  It's an awesome starting point for building commands just like this, with autocompletion, help, bash/zsh support and aliases all built in.

## Helpers and configuration!

Sub has some great conventions, but to allow us to direct commands at any one of our (many) staging and production servers, I forked sub and added some helpers and configuration options.  At HouseTrip, staging servers can be temporary things, so its important we can easily change where to direct commands.

Server connection information lies in a simple yml file for each Rails environment we have.  So we can issue a single command like this;

`ht console staging100`

and be running a Rails console (on the remote staging100 environment) in no time at all.  In the future I'll try to create a pull-request for this (to sub).  Sticking with some conventions, i've written all our ht commands in Ruby and created a number of helper classes for logging output (even talking output with the built-in OSX 'say' command).

## Command Ideas

To give you some ideas, here's a short list of some of the commands we are building (or have already built);

* `ht-ssh (env)` - connect to any server
* `ht-console (env)` - ssh and Rails console onto server
* `ht-dump (db-name)` - grab a fresh (anonymised) database dump and import it to your local env
* `ht-cache-clear (env)` - clear the Rails.cache on a server
* `ht-be-admin (env) (username)` - convert an existing user to an admin
* `ht-jobs (env)` - get some basic stats on the job queues
* `ht-bump-job (env) (id)` - bump priority of a remote job
* `ht-booking (env)` - show stats on the last booking made
* `ht-gif (keyword)` - fetch an animated gif into your paste buffer
* `ht-git-visual (time-ago) (repo)` - visualise git repository activity since (time ago) with [Gource](http://code.google.com/p/gource/)
* `ht-add-server (config)` - add new server details to the ht config file
* `ht-mugshot (keyword)` - grab a photo from our team intranet page
* `ht-starter` - kick off a setup script that installs and configures your laptop for HouseTrip development
