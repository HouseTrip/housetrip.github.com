---
layout: post
title: Efficient cross-process locking in Ruby
published: false
author: Pedro Cunha
author_role: Developer
author_url: http://github.com/pedrocunha
author_avatar: http://www.gravatar.com/avatar/52feffde3f4c3a2fca3e56757f10c269.png
summary: |
  In this blog post we cover how we currently handle sensible transactions
  between several processes in a thread-safe fashion way
---

Baking in thread-safety, and concurrency support in software is both interesting
and a challenge. The problem you want to avoid most of time is shared state.
The truth is: if you ever have to live with it, it will be your bottleneck.

However sometimes that's inevitable, and in the post we will cover one of the
problems we had at HouseTrip and how we solved it.

## The problem

When a host sets up his availability, either by making a property available or
unavailable and someone is trying at the same time book that property for those 
dates, we want to make sure this two events can not happen at the same time.
Since we are in an environment where you have multiple machines each one with 
multiple workers that are single processes, it's not very trivial or assuring 
that a database lock can handle this use-case. Especially when: 
- You have master+replica DB setup and queued jobs reading data from a replica
- Multiple processes using different DB connections, putting the DB under stress
with locks
- You need to lock more than 1 entity at same time
- You need to do run other ruby code that doesn't necessarily need to interact
with the DB. (Writing to mongo or redis for example)

## The solution

The best gracefully way to handle this is by using a remote lock that can be
easily accessed (read + write) by all the processes on your application.
Whenever the process obtains the lock for a specific key, it guarantees you have
exclusive access on that code. Translated to concurrency language, we talking
about a mutex. Only one entity can run inside the exclusive code scope where
others will queue on a FIFO fashion. 

So now, even if our code to book or affect an availability takes a bit longer 
to do (because we are synchronizing processes) we can safely assume certain
operations are definitely atomic! 

We built a gem that transparently provides this feature while it stores either
the lock on a memcache or redis backend. Also it provides features like: 
- Expiration of keys
- Number of retries to get the lock
- Time interval between retries

A code example that initializes as a global variable the lock:

{% highlight ruby %}
# redis = Redis.new
# Or whatever way you have your redis connection
$lock = RemoteLock.new(RemoteLock::Adapters::Redis.new(redis))

def my_method
  $lock.synchronize("some-key") do
    # stuff that needs synchronization in here
  end
end
{% endhighlight %}

In our codebase we go a step further and encapsulate this on a lock class which
allows us to run our lock block code within an ActiveRecord transaction

{% highlight ruby %}
require 'remote_lock'

class Lock
  module ClassMethods
    def acquire(name, wrap_in_transaction = true)
      mutex.synchronize(fixed_name(name)) do
        if wrap_in_transaction
          ActiveRecord::Base.transaction do
            yield if block_given?
          end
        else
          yield if block_given?
        end
      end
    end

    def acquired?(name)
      mutex.acquired?(fixed_name(name))
    end

    private

    def fixed_name(name)
      name.gsub(/\s+/, '-')
    end

    def mutex
      @mutex ||= begin
        redis_adapter =
          RemoteLock::Adapters::Redis.new(RedisConnection)
        RemoteLock.new(redis_adapter, REDIS_LOCK_PREFIX)
      end
    end

  end
  extend ClassMethods
end
{% endhighlight %}

## Conclusions

Thread-safe and concurrency are complex subjects and you should avoid as much as
can dependencies and shared state. However whenever that's not possible there is
a few solutions that can be applied to minimize race conditions. In this post
we presented a way how can achieve a mutex that can shared across processes and
guarantee running mutually exclusive code. 

Also another of the side effect of staying away from DB locks is you can
considerably minimize contention on a database being hammered by writes and
reads every single second. 

You can get our gem through [rubygems](https://rubygems.org/gems/remote_lock)
by putting the following on your Gemfile:

{% highlight ruby %}
gem 'remote_lock'
{% endhighlight %}

You can also check its source code on [github](https://github.com/HouseTrip/remote_lock)
