---
layout: post
title: fuzzily and blurrily: two fast fuzzy-text search/match gems to choose from
published: false
author: Julien
author_url: http://github.com/mezis
author_avatar: http://www.gravatar.com/avatar/88683f31bdf05a8071fb08327b3919cb.png
summary:
---

> Users make spelling mistakes... espacially when typing the name of an
> exotic destination.
>
> Show me properties of **Marakech** !
>
> Here aresome properties in **Marrakesh**, Morroco.
> Did you mean **Martanesh**, Albania, **Marakkanam**, India, or **Marasheshty**, Romania?


[fuzzily](http://github.com/mezis/fuzzily) and
[blurrily](http://github.com/mezis/blurrily) both find misspelled, prefix,
or partial needles in a haystack of strings, quickly.

With a database of 10 million entries, `blurrily` can find fuzzy matches for
any input string within 75ms on typical hardware. On less pathological
datasets, you can easily expect searches to take no more than a few
milliseconds—typically faster than caching!

Both gems are tested with various Ruby VMs and versions of Rails, and we're
actually using `blurrily` in production.

### Fuzzily

The older brother is easier to integrate into an typical Rails application
and lets you make any of a model's attributes searchable:

    class MyStuff < ActiveRecord::Base
      # assuming my_stuffs has a 'name' attribute
      fuzzily_searchable :name
    end

    MyStuff.find_by_fuzzy_name('Some Name', :limit => 10)
    #=> records

Installing it and starting to use it can be done in minutes with typical
Rails applications.

### Blurrily

Fuzzily's younger sibling slightly harder to integrate but it's crazy fast
(it's backed `by a C extension) and scales very well. You can use it as a
client/server:

    $ blurrily &
    $ irb -rubygems -rblurrily/client
    > client = Blurrily::Client.new
    > client.put('London', 1337)
    > client.find('lonndon')
    #=> [1337]

Or directly:

    $ irb -rubygems -rblurrily/map
    > map = 
    > client.put('London', 1337)
    > client.find('lonndon')
    #=> [1337]


If you have a similar problem and feel ElasticSearch of anything Lucene
backed is overkill, give them a go!
