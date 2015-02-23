---
layout: post
title: Bringing Ruby fetch to the Javascript world
published: true
author: Alfredo Motta
author_role: Software Engineer
author_url: http://www.alfredo.motta.name
author_avatar: http://www.gravatar.com/avatar/c1044117a60a9c37a232cf8b6e2c87a8.png
summary: |
    If you are a Rubyist you are probably comfortable using the #fetch method on a day-to-day basis but when you are developing in Javascript this sweetness is not immediately available. This is why I wrote underscorejs-fetch.
---

If you are a Rubyist you are probably comfortable using the [#fetch](http://ruby-doc.org/core-2.2.0/Hash.html#method-i-fetch) method on a day-to-day basis. The gist of its usage is the following:

<pre>
  <code class="ruby">
    item = { name: 'pen', color: 'blue' }
    item.fetch(:name)                                     #=> 'pen'

    item.fetch(:price)                                    #=> key not found error
    item.fetch(:price, '2$')                              #=> '2$'
    item.fetch(:price) { |key| "the #{key.to_s} is 2$"}   #=> "the price is 2$"
  </code>
</pre>

But when you are developing in Javascript this sweetness is not immediately available.
I had a look at [UnderscoreJS](http://underscorejs.org/) but I couldn't find anything similar. [&#95;.has()](http://underscorejs.org/#has), [&#95;.findKey()](http://underscorejs.org/#findKey) or [&#95;.property()](http://underscorejs.org/#property) are not quite what I am looking for.

So I went ahead and I [implemented it myself](https://github.com/mottalrd/underscore-fetch). Here is how it works. If you have  an object and you want to **fetch** an existing property, you simply get it:

<pre>
  <code class="javascript">
    var item = { name: 'pen', color: 'blue' };
    _.fetch(item, 'name'); //=> pen
  </code>
</pre>

On the other hand if the property is not available, you get an *Attribute not found* error in return:

<pre>
  <code class="javascript">
    var item = { name: 'pen', color: 'blue' };
    _.fetch(item, 'price'); //=> Error('Attribute not found');
  </code>
</pre>

Now the sweetest part of *fetch*. When the property you are asking can't be found, you can simply pass a default:

<pre>
  <code class="javascript">
    var item = { name: 'pen', color: 'blue' };
    _.fetch(item, 'price', '2$'); //=> '2$'
  </code>
</pre>

Finally if you are a of sophisticated person you also have the option to pass a callback to determine the default value ending up with something like this:

<pre>
  <code class="javascript">
    var item = { name: 'pen', color: 'blue' };
    _.fetch(item, 'price', function(key) { 
      return "The " + key + " cannot be found"; 
    });
    //=> 'The price cannot be found'
  </code>
</pre>

These example are also available as Jasmine specs in the [Github repository](https://github.com/mottalrd/underscore-fetch/blob/master/spec/underscore-fetch_spec.js). Running the specs is super-easy, just checkout the repository and run:

<pre class="lang:sh decode:true crayon-selected">
  <code>
    sudo npm install -g grunt-cli
    npm install
    grunt jasmine
  </code>
</pre>

There is no reason to publish a 33 lines method function apart from sharing it and having the opportunity to discuss it with the community. 

You love fetch? Then go fetch [the repo on Github](https://github.com/mottalrd/underscore-fetch) and tell me what you think!
