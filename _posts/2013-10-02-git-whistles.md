---
layout: post
title: "Interview: git-whistles gem"
published: true
author: David Silva
author_role: Developer
author_url: https://github.com/davidslv
author_avatar: http://www.gravatar.com/avatar/6aec36daee2fcb518971daa7f2e0f544.png
summary:  |

  Short interview with Developer about one of the tools we use at HouseTrip.

---

  Interviewer: **Salim Hamid**, interviewed: **David Silva**

___Salim Hamid: Hi David, introduce yourself to our readers.___

**David Silva**: Hey, I'm David Silva, and I work as a developer at HouseTrip.

___SH: The git-whistles gem allows us at HouseTrip to enhance our developers’ workflow, 
what was life like before [git-whistles](https://rubygems.org/gems/git-whistles)?___

**DS**: Work was a little slower; creating a branch associated with a story in pivotal tracker 
(an agile work tracking application) was a more tedious process, 
where we needed to input our team name, story name, and story ID to be able to begin work on a feature.

__SH: How did this affect productivity?__

**DS**: Regarding our workflow here at HouseTrip, it was hard to know which branch corresponds 
to which story just by looking at the name of the branch.
This meant we spent too much time trying to locate the relevant stories.


__SH: How did git-whistles solve the problem?__

**DS**: [git-whistles](https://rubygems.org/gems/git-whistles) uses additional tools such as git-select, which allows us to input the story ID and automatically provide us with the correct branch. 
git pivotal-branch creates the name of the branch for us according to HouseTrip conventions (such as team-name/name-of-feature-story-id).
It also creates a comment on pivotal tracker which starts a story for the developers to continue with.
This was really helpful when we were performing tasks that other developers carried on with.

[screen1]()
[screen2]()

__SH: And how did HouseTrip as a whole contribute towards the development of git-whistles?__

**DS**: Julien, our VP of Engineering, actually created the tool initially. 

Pedro, a lead developer here, added his own tweaks and implemented a couple of new features 
such as “git select” and “git latest pushes” as well as “git outstanding-features” 
which are all commands that make processes faster and helps us to practice Agile in a greater way!

Daniel Cruz, one of our developers, created the “pivotal-branch” command.


__SH:What’s your favourite feature of git-whistles?__

**DS**: For me, the one that I like the most is “git select”. 
Say for example I was working on a project on Friday, and then I come in 
on Monday to work on something different, which I was working on previously. 
All I need to do is open Pivotal Tracker, and find the story, get the story ID, 
and use the “git select” command with the story ID to pick up the correct branch. 
This process was a lot more time consuming before git-whistles, because you 
actually had to remember the specific project you were working on all that time ago.

__SH:Who are users of the tool you've built? Did you take into account their opinions while developing it?__

**DS**: Most of the HouseTrip geeks use this tool all the time. 
We needed something that would allow the team to get Agile right, 
and [git-whistles](https://rubygems.org/gems/git-whistles) makes the whole process faster, smoother and indeed more agile!

__SH:How did you measure success? And on that note how successful was the product?__

**DS**: Measuring the success of this project was for us, judged by how much faster we got things done. 
But we encountered huge immediate success on a larger scale, and to date over 7,000 non HouseTrip users have downloaded [git-whistles](https://rubygems.org/gems/git-whistles). 
That’s a success where we’re concerned.

__SH: I’d definitely agree with that. Thanks for your time David and the insight into the development of git-whistles.__

**DS**: Thank you, it was my pleasure.
