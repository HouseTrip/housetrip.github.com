---
layout: post
title: Rspec 1.3.x cookie issue with Ruby 1.9.3 & Rails 2.3.x
published: true
author: Nasir
author_role: Developer
author_url: http://github.com/nas
author_avatar: http://www.gravatar.com/avatar/24880024456ba440c55abbd0dce2c2ed.png
summary: At HouseTrip we use RSpec and cucumber for all our tests. While upgrading our app to Ruby 1.9.3 we hit a strange issue testing cookies in our controller specs.<br/><br/>In RSpec, you can fetch cookie values by using the cookies hash as `cookies[:cookie_name]` just like in the Rails controller, views and helpers. However, all our cookie related tests failed because the value of `cookie_name` was never being set. We found that we could still access the cookies using `response.template.cookies` meaning the cookies were being set correctly, but there was something not right with the tests.
---
At HouseTrip we use RSpec and cucumber for all our tests. While upgrading our app to Ruby 1.9.3 we hit a strange issue testing cookies in our controller specs.

In RSpec, you can fetch cookie values by using the cookies hash as `cookies[:cookie_name]` just like in the Rails controller, views and helpers. However, all our cookie related tests failed because the value of `cookie_name` was never being set. We found that we could still access the cookies using `response.template.cookies` meaning the cookies were being set correctly, but there was something not right with the tests.

{% highlight ruby %}
  response.template #=> ActionView::Base
  response.template.cookies #=> ActionController::CookieJar
{% endhighlight %}

Using `response.template.cookies` in specs presented three issues:

* It did not look as natural as you access cookies in controllers or even in RSpec generally
* While we changed `cookies[:cookie_name]` on our 1.9.3 branch, other developers could still be adding `cookies[:cookie_name]` tests on our 1.8 branch
* An itch to find out the root cause of the issue

We poked into the [Rails cookie tests code](https://github.com/Rails/Rails/blob/2-3-stable/actionpack/test/controller/session/cookie_store_test.rb) and it has a bunch of tests that uses the cookie Hash notation so our tests should have worked.

According to the tests present in `CookieStoreTest`, `headers['Set-Cookie']` could be [nil](https://github.com/Rails/Rails/blob/2-3-stable/actionpack/test/controller/session/cookie_store_test.rb#L185) or a [String](https://github.com/Rails/Rails/blob/2-3-stable/actionpack/test/controller/session/cookie_store_test.rb#L204)

So we grepped/acked the rspec-Rails code for cookies.

* First Suspect: [CookiesProxy class](https://github.com/dchelimsky/RSpec-Rails/blob/v1.3.4/lib/spec/Rails/example/cookies_proxy.rb) does a `require "action_controller/cookies"` which basically provides the ability to set/access/delete cookies in RSpec, not very helpful but it did give us a clue.

* Second Suspect: [FunctionalExampleGroup class](https://github.com/dchelimsky/RSpec-Rails/blob/v1.3.4/lib/spec/Rails/example/functional_example_group.rb) has an instance method called cookies that returns the `CookiesProxy` object. More importantly `FunctionalExampleGroup` inherits [ActionController::TestCase](https://github.com/Rails/Rails/blob/2-3-stable/actionpack/lib/action_controller/test_case.rb) and includes the [ActionController::TestProcess module](https://github.com/Rails/Rails/blob/2-3-stable/actionpack/lib/action_controller/test_process.rb) This has a method called `cookies` that actually returns a cookies hash with cookie name and values. In Rails, ActionController::TestProcess#cookies is implemented as:

{% highlight ruby %}
  def cookies
    cookies = {}
    Array(headers['Set-Cookie']).each do |cookie|
      key, value = cookie.split(";").first.split("=").map {|val| Rack::Utils.unescape(val)}
      cookies[key] = value
    end
    cookies
  end
{% endhighlight %}

It generates the cookie hash using Kernel#Array (mind you, this is not the array class) method to split the cookies from `headers['Set-Cookie']`

For the inquisitive, according to Ruby 1.8 doc:

{% highlight bash %}
  ----------------------------------------------------------- Kernel#Array
     Array(arg)    => array
  ------------------------------------------------------------------------
     Returns _arg_ as an +Array+. First tries to call _arg_+.to_ary+,
     then _arg_+.to_a+. If both fail, creates a single element array
     containing _arg_ (unless _arg_ is +nil+).

        Array(1..5)   #=> [1, 2, 3, 4, 5]

{% endhighlight %}

In Rails 2.3.x, various cookie information is stored in `headers['Set-Cookie']` and is delimited by "\n" where each cookie is composed of several components like;

{% highlight bash %}
  page_views=1;path=/;expires=Fri,10-May-2013 07:27:02 GMT
{% endhighlight %}

Here cookie name is page_views and its value is 1, with other information delimited by `;`

So, it looked like the problem is somewhere in the above `ActionController::TestProcess#cookies` method. Hence, after some debugging exercise, we zeroed in on the `Kernel#Array` method.

In Ruby 1.8

{% highlight ruby %}
  Array("adf\nsadf")
  => ["adf\n", "sadf"]
{% endhighlight %}

and in Ruby 1.9+

{% highlight ruby %}
  Array("adf\nsadf")
  => ["adf\nsadf"]
{% endhighlight %}

Bingo, to illustrate:

Say in Ruby 1.8 the `headers['Set-Cookie']` is set to `page_views=1;path=/;expires=Fri,10-May-2013 07:27:02 GMT\nuser=john;path=/;expires=Fri,10-May-2013 07:27:02 GMT` then doing;

{% highlight bash %}
  >> Array("page_views=1;path=/;expires=Fri,10-May-2013 07:27:02 GMT\nuser=john;path=/;expires=Fri,10-May-2013 07:27:02 GMT")
  => ["page_views=1;path=/;expires=Fri,10-May-2013 07:27:02 GMT\n", "user=john;path=/;expires=Fri,10-May-2013 07:27:02 GMT"]
{% endhighlight %}

will change that into a two element array and rest of the `ActionController::TestProcess#cookies` code will iterate over it and return cookies hash with two keys i.e. page_views and user.  With their respective values and discarding everything after the ; for each cookie.

However, in Ruby 1.9

{% highlight bash %}
  Array("page_views=1;path=/;expires=Fri,10-May-2013 07:27:02 GMT\nuser=john;path=/;expires=Fri,10-May-2013 07:27:02 GMT")
  => ["page_views=1;path=/;expires=Fri,10-May-2013 07:27:02 GMT\nuser=john;path=/;expires=Fri,10-May-2013 07:27:02 GMT"]
{% endhighlight %}

This does not split the string based on `\n` and returns just a single element array. Hence, the `ActionController::TestProcess#cookies` method implementation will just set the first cookie, i.e. page_views value in the cookie hash and discard everything after the `;`.

This whole exercise lead us to our simple solution, which was to slightly change the implementation of the method to;

{% highlight ruby %}
  ...
  header_cookie_list = case headers['Set-Cookie']
  when String
    headers['Set-Cookie'].split("\n")
  when Array
    headers['Set-Cookie'].map{ |x| x.split("\n") }
  end
  Array(header_cookie_list.flatten).each do |cookie|
...
{% endhighlight %}

IMHO it is pointless to use obscure functionality on a method or object that is not obvious or does not make sense, i.e. Kernel#Array to split a string by `\n`.  Neither the name nor the docs suggest anything like that. On the other hand, I am quite content that this has changed in Ruby 1.9+ and similar things like that e.g.

{% highlight ruby %}
  Ruby 1.8
  >> [1,2].to_s
  => "12" # WHY
 
  Ruby 1.9
  [1,2].to_s
  => "[1, 2]"

  Ruby 1.8
  "asdf".to_a
  => ["asdf"] # WHY
  
  Ruby 1.9
  "adf".to_a
  NoMethodError: undefined method `to_a' for "adf":String   # I am quite happy with this
{% endhighlight %}

that people end up misusing.

On another note, Strings are no longer Enumerable in 1.9+ (since they are now encoded). To iterate over a string you need to tell Ruby exactly what you want to iterate on i.e. String#lines, String#chars, String#bytes, etc. For example,

{% highlight ruby %}
  Ruby 1.9

 "adf".chars.to_a
 => ["a", "d", "f"] # which makes lot more sense (same as Ruby 1.8)
{% endhighlight %}

After a little rant, I also feel obliged to mention that I am thankful to all the people who put enormous amount of effort towards Ruby as well as Rails and contributing to make this better.
