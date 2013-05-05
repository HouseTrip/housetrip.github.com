---
layout: post
title: fuzzily and blurrily - two fast fuzzy-text search/match gems
published: true
author: Julien
author_role: Senior Developer
author_url: http://github.com/mezis
author_avatar: http://www.gravatar.com/avatar/88683f31bdf05a8071fb08327b3919cb.png
---

Users make spelling mistakes... espacially when typing the name of an exotic
destination.

{% highlight bash %}
  Show me properties of **Marakech** !

  > Here are some properties in **Marrakesh**, Morroco.
  > Did you mean **Martanesh**, Albania, **Marakkanam**, India, or **Marasheshty**, Romania?
{% endhighlight %}

[fuzzily](http://github.com/mezis/fuzzily) and
[blurrily](http://github.com/mezis/blurrily) both find misspelled, prefix,
or partial needles in a haystack of strings, quickly.

With a database of 10 million entries, `blurrily` can find fuzzy matches for
any input string within 75ms on typical hardware. On less pathological
datasets, you can easily expect searches to take no more than a few
milliseconds, typically faster than caching!

Both gems are tested with various Ruby VMs and versions of Rails, and here at
HouseTrip we're actually using `blurrily` in production.

## Fuzzily

Fuzzily is Blurrily's older brother. It is easier to integrate into an typical
Rails application and lets you make any model's attributes searchable:

{% highlight ruby %}
  class MyStuff < ActiveRecord::Base
    # assuming my_stuffs has a 'name' attribute
    fuzzily_searchable :name
  end

  MyStuff.find_by_fuzzy_name('Some Name', :limit => 10)
  #=> records
{% endhighlight %}

Installing and using it can be done in minutes with a typical Rails application.

## Blurrily

Fuzzily's younger sibling is slightly harder to integrate, but it's crazy fast
(backed by a C extension) and scales very well. You can use it as a
client/server like so:

{% highlight bash %}
  $ blurrily &
  $ irb -rubygems -rblurrily/client
  > client = Blurrily::Client.new
  > client.put('London', 1337)
  > client.find('lonndon')
  #=> [1337]
{% endhighlight %}

Or directly:

{% highlight bash %}
  $ irb -rubygems -rblurrily/map
  > map =
  > client.put('London', 1337)
  > client.find('lonndon')
  #=> [1337]
{% endhighlight %}

If you have a similar problem and feel ElasticSearch or anything Lucene
backed is overkill, why not give them a go!
