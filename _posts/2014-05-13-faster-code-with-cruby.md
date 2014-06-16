---
layout: post
title: Extending Ruby with C for a better perfomance
published: false
author: David Silva
author_role: Associate Software Engineer
author_url: https://github.com/davidslv
author_avatar: http://www.gravatar.com/avatar/6aec36daee2fcb518971daa7f2e0f544.png
summary: |
  I want to share with you the possibility of making your ruby code perform faster by using C extensions to accomplish it.
---

On the 12th of May I attended the [Scottish Ruby Conference](http://2014.scottishrubyconference.com/), and one of the talks that made me realise how much we can still learn and improve was [Introduction to CRuby source code](http://programme2014.scottishrubyconference.com/schedule#proposal_121) by [Andy Pliszka](https://twitter.com/AntiTyping) where he shared how we can improve the performance of our ruby applications by using C Extensions.

In his presentation he **strongly disapproves** messing with the Ruby language source code, which I completely agree with if you are adding a specific functionality for your needs.

Although this still opens a window to improve the performance of your ruby code by exporting your code to use C extensions, where the perfomance boost can be between ~10x and ~50x times faster than your ruby implementation.

## How?

You can achieve this, without touching the source code of the Ruby language, by writting a gem which uses C extensions to solve your specific problem. Building a gem is in my opinion the safest way of continuing upgrading your ruby version in production environment and still keep your code, this way you guarantee that your gem does one thing and does it well without compromising the upgrade of your ruby in production environment.

## But at what cost?

I do believe that most of us are probably a bit rusty in what concerns using C, you will probably spend *maybe* 5x more time building your ruby with C extensions, because in the end you will still program in C but once you get *in the zone* you might end up paying off all the time you spent building it. Andy's presentation shows that we can have a performance gain of more than 10 times, imagine that it means you can take like 5 servers from production, it means you can save money and still process your data faster.

## Where do you start?

I did a quick *Hello World* class just to feel how it works, and honestly it took me around 30 minutes to understand the very basic and make it work in irb. It seems to me that there are lots of tutorials online where they teach you how to start but they don't teach you much, they just give you the feeling of how it works. I personally feel that there is a lack of online documentation as we have with [Ruby-Doc](http://ruby-doc.org/), although I manage to get some references which I wrote at the bottom of this article to help you find the information you need if you ever decide to take this approach.

**My advice is**: try and read as much code as you can from the source of the Ruby language, then write your functionality in ruby code and test it, once you are happy with it, start your journey porting your code to use C extensions.


References:

- [Andy @ LA Ruby Conf 2014](https://www.youtube.com/watch?v=Chk9c8EwrCA)
- [The Ruby README.EXT](https://raw.githubusercontent.com/ruby/ruby/trunk/README.EXT)
- [Rubygems guide](http://guides.rubygems.org/gems-with-extensions/)
- [Stackoverflow](http://stackoverflow.com/tags/ruby-c-api/info)
- [Ruby Hacking Guide](http://rhg.rubyforge.org/)
- [Wrapping up a C library for Ruby](http://blog.firmhouse.com/wrapping-up-a-c-library-for-ruby-it-s-actually-pretty-easy)
- [Extending Ruby with C by Garrett Rooney](http://people.apache.org/~rooneg/talks/ruby-extensions/ruby-extensions.html)
- [Wikibooks C Extensions](http://en.wikibooks.org/wiki/Ruby_Programming/C_Extensions)
- [Packaging Ruby C extensions in nice gems](http://blog.x-aeon.com/2012/11/28/packaging-ruby-c-extensions-in-nice-gems/)
