---
layout: post
title: Handling transactions between processes
published: false
author: Pedro Cunha
author_role: Developer
author_url: http://github.com/pedrocunha
author_avatar: http://www.gravatar.com/avatar/52feffde3f4c3a2fca3e56757f10c269.png
summary: |
  In this blog post we cover how we currently handle sensible transactions between several processes in a tread-safe fashion way
---

One of the most interesting topics on programming is definitely how can you bake a system
that can support concurrency and be thread safe. The problem you want to
avoid most of time is shared state. The truth is: if you ever have to live with it, it will
be your bottleneck.

However sometimes thats inevitable, and in the post we will cover one of the problems we had
at HouseTrip and how we solved it.

## The problem

When an host is setting up his availability, either by making a property available or unavailable
and someone is trying at the same time book that property for those dates, we want to make sure this
two events can not happen at the same time. Since we are in an enviroment where you have multiple machines
each one with multiple workers that are single processes, its not very trivial or assuring that a database lock
can handle this use-case. Especially when: 
- You have master+replica DB setup and a queued jobs reading data from a replica
- Multiple processes using different DB connections, putting the DB under stress with locks
- You need to lock more than 1 entity at same time
- You need to do run other ruby code that doesn't necessarily need to interact with the DB. (Writing to mongo or redis for example)

## The solution

The best gracefully way to handle this is by using a remote lock that can be easily accessed (read + write) by all
the processes on your application. Whenever the process obtains the lock for a specific key, it guarantees you have
exclusive access on that code. Translated to concurrency language, we talking about a mutex. Only one entity
can run inside the exclusive code scope where others will queue on a FIFO fashion. 

So now, even if our code to book or affect an availability takes a bit longer to do (because we are synchronizing
processes) we can safely assume certain operations are definitely atomic! 

We built a gem that transparently provides this feature while it stores either the lock on a memcache or 
redis backend. Also it provides features like: 
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

You can get our gem through [rubygems](https://rubygems.org/gems/remote_lock) by putting the following on your Gemfile:

{% highlight ruby %}
gem 'remote_lock'
{% endhighlight %}

You can also check it's source code on [github](https://github.com/HouseTrip/remote_lock)
