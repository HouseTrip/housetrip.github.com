---
layout: post
title: Maintaining Code Quality at Housetrip
published: true
author: Nasir
author_role: Senior Developer
author_url: http://github.com/nas
author_avatar: http://www.gravatar.com/avatar/24880024456ba440c55abbd0dce2c2ed.png
summary: I joined HouseTrip in August, 2012 since then I have not seen any code deployed to production without a peer review (barring a few exceptional hot-fixes). I found out that the situation was not always like this, in fact, (like many other early stage startups) code used to go live without any reviews and we ended up (in some cases) with careless design and accrued technical debt over a period of time.
---

I joined HouseTrip in August, 2012 since then I have not seen any code deployed to production without a peer review (barring a few exceptional hot-fixes). I found out that the situation was not always like this, in fact, (like many other early stage startups) code used to go live without any reviews and we ended up (in some cases) with careless design and accrued technical debt over a period of time.

Leaving aside the design, architecture and code quality, if you get some buggy code in production then it is going to effect the business' bottom line too. In software, it is hard to guarantee zero bugs, but the probability can be reduced by a rigorous review of what code is released. That's exactly what some of us here thought so we added one more step before any merging. To peer review code changes via a PR (pull request).

I had "code" and "review" in my last sentence and I know (from some past experience) that people can be averse to that. In my opinion however, code reviews don't indicate a lack of trust in your fellow developer.  Instead we accept the fact that nobody writes perfect code and everyone can sometimes make a mistake. Code reviews also provide a great opportunity to learn and share with each other.

Code reviews through PRs have improved code quality all round, and we are all learning a lot from each other (especially new starters at HouseTrip).  Since we are an opinionated bunch (and sometimes question everything) PRs occasionally dissolve into conflicting arguments, that  delay features from being merged and waste time, frustrating the people involved.

So we've produced some simple guidelines to follow during the process.

### Before issuing a PR

* Do not issue a PR in a hurry. Instead take a break and go through the code once again to keep things clean and tidy and finish any obvious refactoring. This will minimise mistakes that you didn't spot and will reduce the chances of unneeded comments that could have been avoided in the first place.

As Uncle Bob says:
> keep the ground cleaner than you found it

* Do not issue a PR unless you think your story is fully complete, i.e. specs, integration/cucumber tests. Reviewers will pick up on these things. An incomplete PR wastes other people's time and makes them question your credibility.

* Always try to keep  PRs small, ideally max 200 - 400 LOC in changes, if possible 50-100 LOCs or less is better, making it easy to review, spot errors and minimise review comments.

### After issuing a PR

This is the most contentious stage in the life of a PR, if you are not careful you might end up crying, screaming, WTFing, getting hurt and hurting other's feelings. You can avoid some of these heartaches if you keep the following in mind.

* Do not merge until all the comments have been addressed, especially if you have comments from more than one person.  Make sure an agreement has been reached between all reviewers and the author.
* Not always but in some cases developers take comments on their code personally and we advise to keep your ego away.  Comments are feedback are about your code and not about you, better learn a trick or two or incorporate something obvious that you had missed. Also seek to understand the reviewer's perspective.
* Be thankful for the reviewer for taking the time out to review your code.

### When reviewing a PR

* Go at an easy pace because faster is not better. Make sure you have enough time in hand before you start reviewing. If you don't have time, just don't review it, otherwise you will end up annoying the dev who issued the PR with your comments or you will miss some obvious errors, defeating the purpose of the review in the first place
* Do not just look at code styling and white spacing. Although they are part of the review it is too important to overlook code maintainability and robustness. Suggest if a test case is missing or if something has been over tested.
* Though we rarely see it now, we used to have one line comments on PRs like, "WTF???" or "whoaaaa" which I am sure the reviewer didn't have any bad intention but later became a major bone of contention. We now think it is better to avoid sarcasm or making demands instead suggest a better approach or offer an alternative. Also when suggesting, do give an example. Ask the author what they think about it or if they have any objections because you don't know what was going on in their head when they wrote it. This doesn't mean suger-coating every comment with smileys. The point is to keep your comment concise and to the point.
* Remember you are giving a feedback or most probably clarifying things that you are not sure about. Your purpose is to find defects and issues but never to show that someone is inferior (or you are a ninja or a rockstar). If you really want to do that, do it in private and most probably you will have your arse kicked (so be ready for that).
* Communicate your ideas clearly. Find ways to make code better. Be humble, explicit and if required or possible then talk to the author in person to clarify things.

Even after all this care, there may still be occasions when disagreements arise.  As developers we love to debate every piece of work that has some business value attached to it, so it is good know when to continue or stop the debate and accept one point of view. If required take the discussion offline and make a decision quickly. If you can't decide among yourselves easily then bring in some mediation and go with the opinion supported by the majority. You might have a solid reason to believe your opinion is correct, but if it is not shared by most of the people around then there are usually two possibilities; either you are wrong or if you are so convinced about your idea you need to go convince others.

We noticed developers often get bogged down in small things like indentation, max chars per line, when to split hashes on multiple lines, hash key/value indentation and a whole bunch of small things. To counter that we  have taken another step to reduce this back and forth, style guides. Our style guides are not fixed and can be changed if a consensus is made. Again the changes on style guides are via PRs. These style guides are forked from other repositories on GitHub that were aligned with our opinions.

* [Ruby Style Guide](https://github.com/HouseTrip/ruby-style-guide)
* [CSS Style Guide](https://github.com/HouseTrip/css-style-guide)
* [Rails Style Guide(very early stage)](https://github.com/HouseTrip/rails-style-guide)

The guides are public and you are welcome to issue PRs against them ;)

We are fully aware that this process may not give optimal results in every team but it has been working quite well at Housetrip with a team of over 20 devs and a good mix of skills (some located locally in London and some in Germany).   Combined with other practices these help us all learn, grow professionally and encourage open communication within our teams and keeping our code base sane and bug free.

P.S. We are growing fast and if you are looking to work on some challenging ideas then [head here](http://hire.jobvite.com/Jobvite/Job.aspx?keywords=Ruby&o=34&j=oqRWWfwT&c=qqb9Vfwp)