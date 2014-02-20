---
layout: post
published: false
title: Quick tip. Is my process alive? (in Ruby)
author: Julien Letessier
author_role: Developer
author_url: http://github.com/mezis
author_avatar: http://www.gravatar.com/avatar/88683f31bdf05a8071fb08327b3919cb
summary: |
  How do you figure wether a process is currently running, in a reasonably portable manner? I keep forgetting how to do this for some reason, and needed it for [my tinkering with Babushka](https://github.com/HouseTrip/babushka-deps/blob/master/erb/kalinka.erb) so might as well write it down.
---

How do you figure wether a process is currently running, in a reasonably
portable manner?
I keep forgetting how to do this for some reason, and needed it for [my
tinkering with
Babushka](https://github.com/HouseTrip/babushka-deps/blob/master/erb/kalinka.erb)
so might as well write it down.

Let's assume you're writing a script that needs to run regularly, but may take a
variable amount of time to run, and you abolutely do not want to run in
parallel.

A typical pattern is to make sure your script writes a "pidfile" containing your
process identifier in a well-known location when starting:

{% highlight ruby %}
  Pathname.new('~/.shebam.pid').write(Process.pid)
{% endhighlight %}

How do you figure out whether a previous version is still running? Just read the
pidfile of course, which leaves you with a handy integer.
You could parse the output of [`ps(1)`](http://www.manpages.info/linux/ps.1.html), but that varies fairly wildly between Unix flavours. 
You could look it up in `/proc`, but that's Linux-only.

It turns out that [standard
Unix](http://pubs.opengroup.org/onlinepubs/9699919799/functions/kill.html) can
help us out. Killing a process with signal 0 (the "null" signal) will perform
error-checking but not send a signal.

A method being worth a thousand words:

{% highlight ruby %}
  module Process
    def exist?(pid)
      Process.kill(0, pid)
      true
    rescue Errno::ESRCH
      false
    end

    module_function :exist?
  end
{% endhighlight %}


A bit of wrapping and you're good to go.
Here's a [working
example](https://github.com/HouseTrip/babushka-deps/blob/f6c016eff2cd9ec7dae846c8eabf4de45f10ae17/lib/single_run_protector.rb).
Still has a race condition, but it's now a pretty small window.
