---
layout: post
title: Maintaining Session Data from Ruby 1.8.x in 1.9.x
published: true
author: Marcus
author_role: Developer
author_url: http://github.com/marcusleemitchell
author_avatar: http://www.gravatar.com/avatar/c0afdc1c900a8f7b429f786507e19758?s=36
summary: tl;dr - You cannot just update to Ruby 1.9.x without losing your session data. Unless you serialize in a portable format (e.g. JSON) and rebuild any complex objects being stored.
---

> tl;dr - You cannot just update to Ruby 1.9.x without losing your session data. Unless you serialize in a portable format (e.g. JSON) and rebuild any complex objects being stored.

### The Problem

Our main application is a bit cumbersome and took us a while to upgrade from Ruby 1.8.7 to Ruby 1.9.x so if you're late to the party as well, hopefully this can help with one aspect of the change.

I'm writing my notes and findings just in case there's anyone who might still want to do this. So, here are the problems. 

* Marshal data has changed enough between Ruby versions so as to be incompatible. (The Ruby core team made no promises that I know of to preserve this data across versions.)
* We need to keep the sessions while updating to Ruby 1.9.x and NOT log users out
* We serialize complex objects that need to keep their type and state once deserialized in order for them to work
* We need to plan for the worst and have new sessions be backward compatible in case of a rollback

Here's a small example to highlight the change in output.  Create a marshalled string in Ruby 1.8.7

{% highlight ruby %}
> RUBY_VERSION
=> "1.8.7"

> Marshal.dump(Base64.encode64("Hi! I'm a string"))
=> "\004\b\"\036SGkhIEknbSBhIHN0cmluZw==\n"
{% endhighlight %}

Compare that to the same string marshaled in Ruby 1.9.3

{% highlight ruby %}
> RUBY_VERSION
=> "1.9.3"

> Marshal.dump(Base64.encode64("Hi! I'm a string"))
=> "\x04\bI\"\x1ESGkhIEknbSBhIHN0cmluZw==\n\x06:\x06EF"
{% endhighlight %}

Trying to load the 1.8.7 created string in 1.9.3 results in an incompatible marshal file format error

{% highlight ruby %}
> Marshal.load(Base64.decode64("\004\b\"\036SGkhIEknbSBhIHN0cmluZw==\n"))
TypeError: incompatible marshal file format (can't be read)
	format version 4.8 required; 30.34 given
	from (irb):5:in `load'
	from (irb):5
	from /Users/mmitchell/.rbenv/versions/1.9.3-p392/bin/irb:12:in `<main>'
{% endhighlight %}

These examples simply serve to show that there is a difference in the output in Ruby 1.8.x and Ruby 1.9.x when using Marshal data.

#### So on to our sessions.

The easiest path would be to ditch all the existing sessions, log all users out of the application, upgrade the application and let everyone log back in. Unfortunately, keeping logged in users logged in while the upgrade took place was one of the most important criteria of this part of the work.

There's also the disaster mitigation to consider…the fact that we might have to roll back and still maintain backwards compatibility for logins created post upgrade (Ruby 1.9) in Ruby 1.8.

#### Possible Solutions to we considered

* Create a script to dump all sessions out to a file. Periodically rerun to keep an un-marshaled, unencoded dump of all session data.

-- No. We currently have 10s of millions of sessions. Nobody in their right mind would consider dumping that much data and try to maintain it.

* Store the session data as YAML with the ability to read both Marshal and YAML and write sessions back as YAML

-- It will certainly work. Backwards compatibility will be maintained, complex objects can be stored as they are with Marshal data. Really quite slow though.

* Store the session data JSON using [Oj](https://github.com/ohler55/oj) (fast JSON parsing lib written in C)

-- Same compatibility benefits as YAML although much faster (benchmarks below). Lose the ability to store complex objects, only keys and values.

Take a look at the benchmarks [here](http://www.ohler.com/dev/oj_misc/performance_strict.html)

> Given the difference in speed and the compatibility…we went with Oj.

---

#### Firstly, let's tackle the backwards compatibility bit

{% highlight ruby %}
def marshal(data)
  Base64.encode64(Oj.dump(data)) if data
end

def unmarshal(data)
  begin
    @decoded_data = Marshal.load(ActiveSupport::Base64.decode64(data))
  rescue
    @decoded_data = Oj.load(ActiveSupport::Base64.decode64(data))
    ...
  ensure
    return @decoded_data || {}
  end
end
{% endhighlight %}
  
This part was relatively easy and as such was taken care of first and forgotten about.
  
Simply put, the read is tried assuming `Marshal` data to read existing sessions, the result is a `Hash` object with whatever values (including complex objects) were originally serialized.
  
Should this fail then the data is assumed to be encoded JSON as written in the `#marshal()` call. All data is to be written as encoded JSON.

This way original sessions are read as Marshal data, written as JSON. New sessions are read as JSON and written as JSON. This will continue to work under Ruby 1.8.7, will work just fine when we make the switch to Ruby 1.9.x and would continue to work were to ever to have to revert to 1.8.7

---

#### Now that's out of the way, let's take a look at these complex objects

#### FlashHash

The most obvious is `ActionController::Flash::FlashHash`(`ActionDispatch::Flash::FlashHash` in Rails 3+)

When setting the flash with `flash[:notice] = "like a baws"` a `FlashHash` object is stored. When you call `flash[:notice]` it is deserialized as a `FlashHash`, used, marked for deletion upon being displayed, swept and no longer visible on the next page render.

Very easy indeed…almost magical.

Now we're no longer storing `FlashHash`, only the data therein as JSON, when deserializing we get a plain ol' `Hash` object. This doesn't have the magic that `FlashHash` has and so fails at everything it was intended to do.

The solution isn't as obvious as setting `flash[:notice] = session[:flash]` unfortunately as this puts the value back into the session, as a Hash and the extra methods are once again lost.

Here's the trick: we're going to recreate the original `FlashHash` type object from the stored Hash in such a way that it doesn't get put straight back into the session store. This is done by creating a blank `FlashHash` object and then updating it.

We then need to tell it what key has been used.

{% highlight ruby %}
def unmarshal(data)
  begin
    @decoded_data = Marshal.load(Base64.decode64(data))
  rescue
    @decoded_data = Oj.load(ActiveSupport::Base64.decode64(data))
    @decoded_data["flash"] = flash_unmarshal(@decoded_data.delete("flash")) if @decoded_data.has_key?("flash")
  ensure
    return @decoded_data || {}
  end
end

private

def flash_unmarshal(value)
  return unless value
  return value unless defined?(ActionController::Flash::FlashHash)
  
  flash = ActionController::Flash::FlashHash.new
  flash.update(value.with_indifferent_access)
  
  # mark what has been used so that .sweep will remove it
  # sweep happens as an after_filter in applitcation_controller
  used_key = value.delete(value.keys.first)
  flash.discard(used_key)
  
  return flash
end
{% endhighlight %}

The keys here are `ActionController::Flash::FlashHash.new.update` and `flash.discard(used_key)`

and in `application_conrtoller.rb`

{% highlight ruby %}
after_filter :sweep_flash

def sweep_flash
  flash.sweep unless response.status.to_s =~ /302/
end
{% endhighlight %}

As we're manually marking the key for deletion this stops the flash message from being swept if the result of the action is a redirect (rather than a render which should display the message and sweep the used key)

#### OpenStruct

We have a couple of other instances of OpenStructs being stored. These caused a few issues but were easier to solve. Keys know to contain OpenStruct data need to be recast into a meaningful object and allow get/set to any methods defined on them.

It's very easy to define attributes on an OpenStruct on the fly

> Our use case was to preserve drop-down selections in a report generator when the results are returned.

`attributes` are the contents of the report form submission.

Originally:

{% highlight ruby %}
@filter = OpenStruct.new(attributes)
…
report_params = {
  :date_range => @filter.from..@filter.to,
  :user_id => @filter.user.or(nil),
  :angle => @filter.angle
  …
}
{% endhighlight %}

became:

{% highlight ruby %}
class FilterPersistanceHash < HashWithIndifferentAccess
  [some methods for formatting specific data in the views]
end

@filter = FilterPersistanceHash[attributes]
…
report_params = {
  :date_range => @filter[:from]..@filter[:to],
  :user_id => @filter[:user].or(nil),
  :angle => @filter[:angle]
  …
}

{% endhighlight %}

Note the switch from dot syntax gets to Hash-like key selection. This had to be done site wide.

> I did at one point consider a `method_missing` method on `FilterPersistanceHash` to catch all method calls and recall using Hash key syntax but there was too much scope for abuse and poor error handling exposed by this so I ditched it.

Given a fairly comprehensive test suite problems with a change to the session storage were highlighted very quickly. Finding one method that would fit across the whole application given that different complex object types were being serialized…without too many changes to the code and to coding practices was a challenge.

What we have now is code that can go to production ahead of the Ruby 1.9.x update and run. Sessions being used in this time will get converted and be maintained into the future. Those (many) sessions that are not used will have to be ditched. These sessions will be from one time or old users of the site and will simply have to log in once again.