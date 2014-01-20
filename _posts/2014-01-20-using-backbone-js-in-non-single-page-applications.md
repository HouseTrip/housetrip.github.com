---
layout: post
title: Using BackboneJS in non-single-page applications
published: true
author: Jesper Kjeldgaard
author_role: Developer
author_url: http://github.com/thejspr
author_avatar: http://www.gravatar.com/avatar/e96273c9bd8ea77a17ae87dca4c0de4c
summary: |
  During a recent redesign of several essential pages, we decided to create a
  new layout and adopt BackboneJS as our front-end framework. This post describes
  our approach to introducing a JavaScript framework into a large Rails application.
---

When recently tasked with implementing a redesign of several of our essential
pages, we decided to start with a new application layout. This allowed us to
rethink the technologies we use in our front-end stack, which is awesome, but
also carries a great responsibility in making maintainable decisions.

As one of the requirements of the project was to cater to mobile devices, we
decided to use [Twitter Bootstrap](http://getbootstrap.com) as the foundation
for our styles and layout. This gave the teams involved a structure to follow
and many components to extend from, besides this we documented some guide lines
in a [Sass style guide](https://github.com/HouseTrip/sass-style-guide).

During the early stages of the project it also became clear that we needed to
have a solid structure and set of conventions in place for our JavaScript. After
considering the many JS frameworks available we decided to go with
[BackboneJS](http://backbonejs.org).
We chose this technology because it gave us the structure and building blocks
needed to get the job done, without getting in the way too much. We also
considered [AngularJS](http://angularjs.org) and [EmberJS](http://emberjs.com),
but decided against these two as they cater more to a fully JavaScript driven
front-end. By this meaning that we still wanted to do most of the page rendering
on the server and then initialize the JS functionality once the page is loaded.

Once we decided on BackboneJS, we put down the rule that all JavaScript in the
new layout should be wrapped in Backbone so that we don't end up with jQuery
spaghetti years down the line. This decision was a bit controversial to begin
with, but has worked out great in terms of keeping our front-end code consistent
and maintainable across teams.

## Organizing the new layout

Introducing BackboneJS into the application created a new set of decisions to
make in terms of file organization and object naming. We decided to do the
simplest thing that could work, which is bundling all JavaScript files together
into one file and including it on all pages. This increases the load time
when the user first enters the site, but subsequent requests benefit from having the
JavaScript cached. As this file grows in size we might consider using a module
loader like [RequireJS](http://requirejs.org) or
[Browserify](http://browserify.org), but for now we stick to the
[YAGNI](http://en.wikipedia.org/wiki/You_aren't_gonna_need_it) principle.

To avoid naming collisions we agreed the following namespacing structure:

{% highlight javascript %}
window.HouseTrip = window.HouseTrip || {};

HouseTrip.Models = {}
HouseTrip.Collections = {}
HouseTrip.Views = {}
{% endhighlight %}

This initialization is included at the top of our concatenated JavaScript file,
allowing us to attach classes to it as they get loaded. A model would for
instance be defined in a separate file as:

{% highlight javascript %}
HouseTrip.Models.Property = Backbone.Model.extend({})
{% endhighlight %}

This allows us to gather all relevant code in their own namespace, and having
access to any resource anywhere we might need it. We decided to document
these decisions in a [BackboneJS style guide](https://github.com/HouseTrip/backbone-style-guide).

## Final thoughts

I think the decision to go with BackboneJS and do the most of the rendering in
Rails is a great way to transition a large application towards a more modern
and performant front-end. Whether it will still be maintainable 2 years down
the line is hard to say, but it has worked great so far for a project spanning
three teams over several months.
