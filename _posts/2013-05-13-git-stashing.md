---
layout: post
title: Git Stashing
published: true
author: Ignazio
author_role: Developer
author_url: https://github.com/gnagno
author_avatar: https://secure.gravatar.com/avatar/7eb9a418af99971678657c3ba34173f3
summary: Moving between different branches during my work I have discovered how useful git stash can be. The more I use it and the more tricks I read, the more confident I feel with it.  The stash is useful when you need to save your changes to a temporary place, move onto something else and then get back to your changes afterwards.
---

Moving between different branches during my work I have discovered how useful git stash can be. The more I use it and the more tricks I read, the more confident I feel with it.

The stash is useful when you need to save your changes to a temporary place, move onto something else and then get back to your changes afterwards.

### Adding your code changes to the stash

You stash your work by running:

{% highlight bash %}
  git add .
  git stash
{% endhighlight %}

This adds your code modifications to the top of the stash.

### Listing your stashes

To see the list of stashed changes simply run:

{% highlight bash %}
  git stash list
{% endhighlight %}

this will return something like:

{% highlight bash %}
  stash@{0}: WIP on another-feature: fc92b1f initial commit
  stash@{1}: WIP on cool-feature: fc92b1f initial commit
{% endhighlight %}

Every stash is identified by the string `stash@{<number>}`. The lower the number is in the braces, the more recent the stash is.

### Adding more information to the list

So you stashed your changes and saw the list, but let's admit it, the list is a bit cryptic.  What we need is something that allows us to apply a label. This can be achieved using the save parameter:

{% highlight bash %}
  git stash save "something meaningful"
{% endhighlight %}

Listing the stash again, we now have:

{% highlight bash %}
  stash@{0}: On another-cool-feature: something meaningful
  stash@{1}: WIP on another-feature: fc92b1f initial commit
  stash@{2}: WIP on cool-feature: fc92b1f initial commit
{% endhighlight %}

### What did I just save?

What happens when you want to get back your work but you don't remember what you saved in your stash? Well... you can use the show command to see the changes in your stash:

{% highlight bash %}
  git stash show
{% endhighlight %}

And you will get a diff of the changes, by default this will show you a diff between your current working directory and the last stash. If you want to see the diff with an older stash e.g. stash@{2} you can do:

{% highlight bash %}
  git stash show stash@{2}
{% endhighlight %}

### Popping vs applying

Cool you saved your stash, and you saw what you saved, how can you re-apply the change you made?

There are two commands to do this, apply and pop, both will apply the stash to your code. Pop will delete the stash from the list, apply will leave it in place:

{% highlight bash %}
  git stash pop
{% endhighlight %}

And the last stash will be applied to your code. Again if you don't want to apply the last stash, but an earlier one you can specify the stash id with:

{% highlight bash %}
  git stash pop stash@{2}
{% endhighlight %}

### Deleting stashes

When you start to use the stash more, your stashed list may get quite long. To delete a stash (for example the one identified by stash@{2}) you simply run:

{% highlight bash %}
  git stash drop stash@{2}
{% endhighlight %}

And stash@{2} will be gone forever!

If you want to delete everything in your stash use:

{% highlight bash %}
  git stash clear
{% endhighlight %}

### One more trick

Let's suppose you want to create a new branch from your working branch and apply one of your stashes:

{% highlight bash %}
  git checkout original-branch
  git checkout -b my-new-beautiful-branch
  git stash pop stash@{0}
{% endhighlight %}

And you have a new my-new-beautiful-branch with the modifications from stash@{0}.  You can join these two commands in one with:

{% highlight bash %}
  git stash create my-new-beautiful-branch stash@{0}
{% endhighlight %}

It's interesting to note here that you can execute this command from any branch, the resulting branch (my-new-beautiful-branch) will always be a direct branch of original-branch. And if you omit the stash id, this command will use the most recent stashed change by default.

Thats it, happy stashing!
