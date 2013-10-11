---
layout: post
title: Introducing git-whistles
published: true
author: David Silva
author_role: Developer
author_url: https://github.com/davidslv
author_avatar: http://www.gravatar.com/avatar/6aec36daee2fcb518971daa7f2e0f544.png
summary:  |

  A short interview with (Developer) David Silva. The topic of discussion is git-whistles,
  a gem we use extensively here at HouseTrip that helps with our git based workflow.

---

  Interviewer: **Salim Hamid**, Interviewee: **David Silva**

___SH: Hi David, introduce yourself to our readers.___

**DS**: Hey, I'm David Silva, and I work as a developer at HouseTrip.

___SH: The git-whistles gem allows developers at HouseTrip to enhance their workflow,
what was life like before [git-whistles](https://rubygems.org/gems/git-whistles)?___

**DS**: The gem provides us with additional git commands. Without them, work was a little slower.  For example; creating a git feature branch associated with a story in Pivotal Tracker was a more tedious process.  We needed to first input our team name, then decide on an abbreviated story label and finally copy and paste the story ID, all before we could get started with development in the branch.

__SH: How did this affect productivity?__

**DS**: Regarding our workflow here at HouseTrip, it was hard to know which branch corresponded to which story just by looking at its' name. Meaning we were spending too much time locating stories.

__SH: How did git-whistles solve the problem?__

**DS**: [git-whistles](https://rubygems.org/gems/git-whistles) uses additional tools such as `git-select`, we can ask for a story ID and it returns the corresponding story branch name. And `git pivotal-branch` creates the name of the branch for us according to HouseTrip conventions (such as team-name/name-of-feature-story-id). It also creates a comment on pivotal tracker changing the story status to "started". This is really helpful when we perform tasks that other developers must carry on with in the same story.

<img src="/images/2013-10-02/pivotal-command.png" title="pivotal-branch command" alt="pivotal-branch command"/>
<img src="/images/2013-10-02/pivotal-tracker-story.png" title="pivotal tracker story" alt="pivotal tracker story"/>

__SH: And how did HouseTrip as a whole contribute towards the development of git-whistles?__

**DS**: Julien, our VP of Engineering, actually created the tool initially.

Daniel (developer) created the `git pivotal-branch` command. It creates a new feature branch based on the title of a Pivotal Tracker story and its id.  Pedro (lead developer), added his own tweaks and implemented a couple of new features such as `git select`, `git latest-pushes` and `git outstanding-features`, all commands that make working with git faster and help us practice agile story development more effectively!

__SH: What’s your favourite feature of git-whistles?__

**DS**: For me, the command I like the most is `git select`. It make managing multiple local git branches easier.
For example, if I was working on a project on Friday, and then returned to on Monday (now working on something different). All I need to do is open Pivotal Tracker, find the story, get the story ID, and use the "git select" command with the story ID to pick up the correct branch again. This process was a lot more time consuming before git-whistles, because you actually had to remember the full branch name for the project you were working on.

__SH: Who are users of the tool you've built? Did you take into account their opinions while developing it?__

**DS**: Most of the HouseTrip developers use this tool daily. We needed something that would allow the team to get agile right, and [git-whistles](https://rubygems.org/gems/git-whistles) makes the process faster, smoother and indeed more agile!

__SH:How did you measure success? And on that note how successful was the product?__

**DS**: Measuring the success of this project was for us, judged by how much faster we got things done.
But we encountered huge immediate success on a larger scale, and to date [git-whistles](https://rubygems.org/gems/git-whistles) has been downloaded more than 7000 times.
That’s a success where we’re concerned.

__SH: I’d definitely agree with that. Thanks for your time David and the insight into the development of git-whistles.__

**DS**: Thank you, it was my pleasure. You can find out more about the gem and how it works [here on GitHub](https://github.com/mezis/git-whistles)
