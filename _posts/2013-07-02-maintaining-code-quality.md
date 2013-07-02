---
layout: post
title: Maintaining Code Quality at Houstrip
published: true
author: Nasir
author_role: Senior Developer
author_url: http://github.com/nas
author_avatar: http://www.gravatar.com/avatar/24880024456ba440c55abbd0dce2c2ed.png
summary: I joined HouseTrip in August, 2012 since then I have not seen any code going to production without peer review barring a few exceptional circumstances of hotfixes. I found out very early that the situation was not always like that, in fact, like many other early stage startups, code used to go live without any reviews and hence we ended up, in some cases, with not well thought out design and ended up accruing some technical debt over a period of time.
---

I joined HouseTrip in August, 2012 since then I have not seen any code going to production without peer review barring a few exceptional circumstances of hotfixes. I found out very early that the situation was not always like that, in fact, like many other early stage startups, code used to go live without any reviews and hence we ended up, in some cases, with not well thought out design and ended up accruing some technical debt over a period of time.

Leaving aside the design, architecture and code quality, if you get some buggy code in production then it is going to effect the business' bottom line too. In software, it is hard to guarantee no bugs but the probability of putting a bug out in production could be reduced by doing a rigorous review of what goes out. Exactly that's what some of us (I wasn't here then so cannot take any credit for that) at HouseTrip thought and added one more step before any code went out to production, i.e., to peer review the code changes via PR (pull requests). Since I had "code" and "review" in my last sentence and I know from my past experience that some people are averse to that and go "oh you don't trust my code". However, IMHO doing code review doesn't indicate you do not trust your colleagues but we all accept the fact that nobody writes perfect code and everyone makes mistakes, also it is a great opportunity to learn and share with each other .

That additional step did improve the code quality and we all have learnt a lot from each other (especially someone who has recently joined) because of PRs but in some cases devs got into severe conflicting arguments, since we are an opinionated bunch and question everything a lot, that not only delayed the feature getting out but also wasted team's time as well as frustrated the people involved in it.

Hence, we came up with some simple guidelines that we should follow during the whole process.

### Before issuing the PR

* Do not issue a PR in a hurry instead take a break and go through the code once again to keep things clean and tidy or any obvious refactoring you need. This will minimize any obvious mistakes that you didn't spot and will reduce the chances of unneeded comments that could have been avoided in the first place.
As Uncle Bob says:
> keep the ground cleaner than you found it
* Do not issue a PR unless you think your story is fully complete, i.e. specs, integration/cucumber tests. Otherwise others will ask you to do that which could have been avoided in the first place. By doing that you are not only wasting other people's time but also making them question your credibility if you are doing it repeatedly.
* Always try to keep your PRs small, ideally max 200 - 400 LOCs, if possible 50-100 LOCs or less that makes it easy to review, spot errors, minimize comments, etc

### After issuing PR

This is the most contentious stage in the life of a PR and this is where if you are not careful, then you might end up crying, screaming, WTFing, getting hurt and hurting others feelings. You can avoid some of these heartaches if you keep the following in mind.

* Do not merge until all the comments have been addressed, especially if you have comments from more than one person then make sure they all have been taken care of or reached an agreement.
* Not always but in some cases developers take comments on their code personally and we advise to keep your ego away, comments are feedback about your code and not about you, better learn a trick or two or incorporate something obvious that you had missed. Seek to understand reviewers perspective.
* Be thankful for the reviewer for taking the time out as s/he could be doing something else in that time but is reviewing your work and sharing his/her experience.

### When reviewing PR

* Go at an easy pace because faster is not better. Make sure you have enough time in hand before you start reviewing. If you don't have time, then just don't review it otherwise you will either end up pissing off the dev who issued the PR with your comments or you will miss obvious errors which defeats the purpose doing it in the first place.
* Do not just look at code styling, white spaces, etc. Although they are part of the review but it is too important to overlook for code maintainability and robustness. Suggest if a test case is missing or if it is being over tested.
* Though we don't see it any more but in some cases we had comments on PRs as "WTF???" or "whoaaaa" which I am sure that the reviewer didn't have any bad intention to write but became a major bone of contention. We now think it is better to avoid sarcasm or making demands instead suggest. Also when suggesting, give an example, ask the dev what he thinks about it or if he has any objections because you don't know what was going on in his/er head when s/he wrote it. However, that doesn't mean we sugercoat every comment with smileys and please. The point is to keep your comment concise and to the point.
* Remember you are giving a feedback or most probably clarifying things that you are not sure about. Your purpose is to find defects and issues but never to show that someone is inferior or you are a ninja or a rockstar. If you really want to do it, do it in private - and most probably you will have your arse kicked (so be ready for that).
* Communicate your ideas clearly. Find ways to make code better. Be humble, explicit and if required or possible then talk in person, clarify and discuss.

Even after all this care, there may still be occasions for disagreements and as developers we love to debate but every piece of your work has some business value attached to it so it is good know when to continue or stop the debate and accept one or the other point of view. If required take the discussion offline and make a decision quickly. If you can't decide among yourself easily then it is time to bring in some mediation and just go with the opinion supported by the majority. You might have a solid reason to believe in your opinion but if it is not shared by most of the people around then there can only be two possibilities either you are wrong or if you are so convinced about your idea then go and convince the others too.

We noticed developers often get bogged down in small things like indentation, max chars per line, when to split hashes on mutiple lines, hash key/value indentation and a whole bunch of other things. To counter that we recently have taken another step to reduce the back and forth on these small stylistic issues where people have different preferences by rolling out some style guides. Those style guides are not fixed and can be changed if everyone agrees on it. Again the changes on those style guides are done via PRs. These style guides are by no means our own from scratch or complete. We have have forked them from other repos on github that made sense or combined a few together.
* [Ruby Style Guide](https://github.com/HouseTrip/ruby-style-guide)
* [CSS Style Guide](https://github.com/HouseTrip/css-style-guide)
* [Rails Style Guide(very early stage)](https://github.com/HouseTrip/rails-style-guide)

You are welcome to issue PRs against above guides ;)

Finally, we realise there is no best solution for anything but we can go with something better and what makes sense at the point of making that decision. We are fully aware that this process may not give optimal results in every kind of team but it has been working quite well at Housetrip with a team of over 20 devs, with a good mix of skills, some located locally in london and some in Germany. We use this process, in addition to other measures, as one of the means of learning, growing professionally, open communication among and within teams and last but not the least keeping the code sane and bug free.

P.S. We are growing fast and if you are looking to work on some challenging ideas then [head here](http://hire.jobvite.com/Jobvite/Job.aspx?keywords=Ruby&o=34&j=oqRWWfwT&c=qqb9Vfwp)
