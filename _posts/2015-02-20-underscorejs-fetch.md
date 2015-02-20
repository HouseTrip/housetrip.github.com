---
layout: post
title: Bringing Ruby fetch to the Javascript world
published: true
author: Alfredo Motta
author_role: Software Engineer
author_url: http://www.alfredo.motta.name
author_avatar: http://www.gravatar.com/avatar/c1044117a60a9c37a232cf8b6e2c87a8.png
summary: |
    If you are a Rubyist you are probably comfortable using the #fetch method on a day-to-day basis but when you are developing in Javascript this sweetness is not immediately available. Here is why I wrote underscorejs-fetch.
---

If you are a Rubyist you are probably comfortable using the [#fetch](http://ruby-doc.org/core-2.2.0/Hash.html#method-i-fetch) method on a day-to-day basis. The gist of it's usage is the following:

<pre>
  <code class="ruby">
    h = { "a" =&gt; 100, "b" =&gt; 200 }
    h.fetch("a")                            #=&gt; 100
    h.fetch("z", "go fish")                 #=&gt; "go fish"
    h.fetch("z") { |el| "go fish, #{el}"}   #=&gt; "go fish, z"

    h = { "a" =&gt; 100, "b" =&gt; 200 }
    h.fetch("z") #=&gt; prog.rb:2:in `fetch': key not found (KeyError)
  </code>
</pre>

But when you are developing in Javascript this sweetness is not immediately available.
I had a look at [UnderscoreJS](http://underscorejs.org/) but I couldn't find anything similar. [&#95;.has()](http://underscorejs.org/#has), [&#95;.findKey()](http://underscorejs.org/#findKey) or [&#95;.property()](http://underscorejs.org/#property) are not quite what I am looking for.

So I went ahead and I [implemented it myself](https://github.com/mottalrd/underscore-fetch). Here is how it works. If you have  an object and you want to **fetch** an existing property, you simply get it:

<pre>
  <code class="javascript">
    var object = { color: 'blue' };
    _.fetch(object, 'color', 'blue'); //=&gt; blue
  </code>
</pre>

On the other hand if the property is not available, you get an *Attribute not found* error in return:

<pre>
  <code class="javascript">
    var object = { };
    _.fetch(object, 'color'); //=&gt; Error('Attribute not found');
  </code>
</pre>

Now the sweetest part of *fetch*. When the property you are asking can't be found, you can simply pass a default:

<pre>
  <code class="javascript">
    var object = { };
    _.fetch(object, 'color', 'black'); //=&gt; 'black'
  </code>
</pre>

Finally if you are a of sophisticated person you also have the option to pass a callback to determine the default value ending up with something like this:

<pre>
  <code class="javascript">
    var object = { };
    var condition = 1;
    _.fetch(object, 'color', function() { 
      return condition == 1 ? 'black' : 'blue'; 
    });
    //=&gt; 'black'
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

There is no reason to publish a 33 lines method function apart from sharing it and having the opportunity to discuss it with the community. You love fetch? Then go fetch [the repo on Github](https://github.com/mottalrd/underscore-fetch) and tell me what you think!
