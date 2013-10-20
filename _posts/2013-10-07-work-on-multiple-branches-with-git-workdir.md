---
layout: post
title: Work in multiple branches with Git Workdir
published: true
author: Jesper Kjeldgaard
author_role: Developer
author_url: http://github.com/thejspr
author_avatar: http://www.gravatar.com/avatar/e96273c9bd8ea77a17ae87dca4c0de4c
summary: |
  Ever wanted to have your checkout in multiple states at one time without
  having to maintain several `.git` folders? This post described how a script
  from `git-contrib` can help you achieve parallel git nirvana.
---

Sometimes I find the need to work on multiple branches in the same repository at
once. This can be caused by many different scenarios; waiting for tests to
finish, running two versions of an application at the same time etc. In short,
anywhere where a regular `git stash/branch` workflow falls short. I also wanted
a solution where I didn't have to manage multiple `.git` folders.

## Git workdir to the rescue

I found a useful script in
[git-contrib](https://github.com/git/git/tree/master/contrib) called
`git-workdir` which enables having the same repository in multiple states. Since
it's just a bash script, there are many ways to install it, I've done it like
this;

{% highlight bash %}
cd ~/Code/libs/
git clone git@github.com/git/git.git

# add an alias for the workdir script
alias git-new-workdir='~/Code/libs/git/contrib/workdir/git-new-workdir'
{% endhighlight %}

With the script installed, you can now create a 'clone' of your checkout like
this;

{% highlight bash %}
# usage: git-new-workdir <checkout-folder> <clone-folder>

git-new-workdir my-app my-app-CLONE
{% endhighlight %}

The script achieves having multiple states on the same checkout by using some
arcane symlinking which I will not go into detail with here. For more
information about the script, have a look at [the
source](https://github.com/git/git/blob/master/contrib/workdir/git-new-workdir).

## Caveats

Having multiple states on the same checkout can easily create problems if you're
not paying attention. I recommend strong coffee and/or sufficient sleep
beforehand.
