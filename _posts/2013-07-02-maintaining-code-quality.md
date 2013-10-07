---
layout: post
title: Maintaining Code Quality at Housetrip
published: true
author: Nasir
author_role: Developer
author_url: http://github.com/nas
author_avatar: http://www.gravatar.com/avatar/24880024456ba440c55abbd0dce2c2ed.png
summary: I joined HouseTrip in August, 2012 and since then I have not seen any code deployed to production without a peer review (barring a few exceptional hot-fixes). I found out that this was not always the case, in fact, (like many early stage startups) code used to go live without any reviews and we sometimes ended up with careless design and accrued technical debt.
---

I joined HouseTrip in August, 2012 and since then I have not seen any code deployed to production without a peer review (barring a few exceptional hot-fixes). I found out that this was not always the case, in fact, (like many early stage startups) code used to go live without any reviews and we sometimes ended up with careless design and accrued technical debt

Leaving aside the design, architecture and code quality, if you have some buggy code in production then it's going to effect the business' bottom line. In software, it is hard to guarantee zero bugs, but the probability of errors can be reduced by a rigorous review of pre-release code. And that's exactly what some of us here thought.  So we added one more step before any merging can take place. To peer review code changes via a PR (pull request).

I had "code" and "review" in my last sentence and I know (from past experience) that some people can be averse to that. In my opinion however, code reviews don't indicate a lack of trust in your fellow developer.  Instead we can accept the fact that no-one writes perfect code all the time, and anyone can make a mistake. Code reviews also provide a great opportunity to learn and share with each other.

Our code reviews (through PRs) have improved code quality all round, and we are all learning a lot from each other (especially new starters at HouseTrip).  Since we are an opinionated bunch (and sometimes question everything) a PR can occasionally dissolve with conflicting arguments, that end up delaying features and wasting time, frustrating the people involved.

To help with this we've produced some simple guidelines to follow during the process.

### Before issuing a PR

* Do not issue a PR in a hurry. Instead take a break and go through the code once again to keep things clean and tidy and finish any obvious refactoring. This will minimise mistakes that you missed and will reduce the chances of unnecessary comments.

As Uncle Bob says:
> keep the ground cleaner than you found it

* Do not issue a PR unless you think your story is fully complete, i.e. specs, integration/cucumber tests. Reviewers will pick up on these things right away. An incomplete PR wastes other people's time and makes them question your credibility.

* Always try to keep  PRs small, ideally max 200 - 400 LOC in changes, if possible 50-100 LOCs or less is better, making it easy to review, spot errors and minimise comments.

### After issuing a PR

This is the most contentious stage in the life of a PR. If you are not careful you might end up crying, screaming, WTFing, getting hurt or hurting someone else's feelings. You can avoid some of these heartaches by keeping the following in mind;

* Do not merge until all the comments have been addressed, especially if you have comments from more than one person.  Make sure an agreement has been reached between all reviewers and the author.
* Not always but in some cases developers take comments on their code personally and we advise to keep your ego away.  Comments are feedback are about your code and not about you, better learn a trick or two or incorporate something obvious that you had missed. Also seek to understand the reviewer's perspective.
* Be thankful for the reviewer for taking the time out to review your code.

### When reviewing a PR

* Go at an easy pace because faster is not better. Make sure you have enough time in hand before you start reviewing. If you don't have time, just don't review. Otherwise you'll end up annoying the author who issued the PR with your comments or you will miss some obvious errors, defeating the purpose in the first place
* Do not just look at code styling and white spacing. Although they are part of the review it is too important to overlook code maintainability and robustness. Suggest if a test case is missing or if something has been over tested.
* Though we rarely see it now, we used to have one line comments on PRs like, "WTF???" or "whoaaaa!" where the reviewer didn't have any bad intentions but the comment became a major bone of contention. We now think it is better to avoid sarcasm or making demands and instead always suggest a better approach or offer an alternative. When suggesting do give an example. Ask the author what they think or if they have objections; you don't know what was going on in their head when they wrote the code. This doesn't mean sugar-coating every comment with smileys. The point is to keep your comment concise, useful and to the point.
* Remember you are giving feedback or clarifying things you are not sure about. The purpose is to find defects and issues but never show that someone is inferior (or you are a ninja or a rockstar). If you really want to do that, do it in private and most probably you will have your arse kicked (so be ready for that).
* Communicate your ideas clearly. Find ways to make code better. Be humble, explicit and if required (or possible) talk to the author in person to clarify things.

Even after all this care, there may still be occasions when disagreements arise.  As developers we love to debate every piece of work that has some business value attached to it, so it is good know when to continue or stop the debate and accept one point of view. If required take the discussion offline and make a decision quickly. If you can't decide among yourselves easily then bring in some mediation and go with the majority opinion. You might have a solid reason to believe your opinion is correct, but if it's not shared by a majority then there are usually two possibilities; either you are wrong and won't admit it or you need to convince the others better.

We noticed developers often get bogged down in small things like indentation, chars per line, when to split hashes, hash key/value indentation and a whole bunch of small things. To counter that we have taken another step to reduce this back and forth, **style guides**.

Our style guides are not fixed and can be changed if a consensus is made (via PRs). These guides are forked from other repositories on GitHub (that were aligned with our opinions).

* [Ruby Style Guide](https://github.com/HouseTrip/ruby-style-guide)
* [CSS Style Guide](https://github.com/HouseTrip/css-style-guide)
* [Rails Style Guide](https://github.com/HouseTrip/rails-style-guide) (early stage)

The guides are public and you are welcome to issue PRs against them yourself ;)

We are fully aware that this process may not give optimal results in every team, but it has been working quite well at Housetrip with a team of over 20 developers and a good mix of skills (most located here in London and a few remote developers in Germany). Combined with other practices these help us learn, grow professionally and encourage open communication within our teams, while keeping our code base sane and bug free.

PS. We are growing fast and if you are looking to work on some challenging ideas then [apply here](http://blog.housetrip.com/team/housetrip-jobs) today!
