---
layout: post
title: Decoupling JavaScript applications using the Publish/Subscribe pattern
published: true
author: Emili Parreño
author_role: Software Engineer
author_url: http://github.com/eparreno
author_avatar: http://www.gravatar.com/avatar/9b3b1fd6baa8379638d8399ecd60045d.png
summary: |
  Design patterns are a great tool for developers, providing us with generalised
  and reusable solutions to common problems in software development. In this
  post we'll see one of the more valuable patterns for modern applications, the
  Publish/Subscribe pattern and how to use it in JavaScript applications.
---

Design patterns are a great tool for developers, providing us with generalised
and reusable solutions to common problems in software development. In this
post we'll see one of the more valuable patterns for modern applications, the
Publish/Subscribe pattern and how to use it in JavaScript applications.

## The problem
Let me use one of the most common features in modern web as to introduce the
problem: sending notification emails.
Let's say we're building an e-commerce site and we'd like to send a
notification email to the customer when they make a purchase. A simple
solution, and probably the most common one, could be something like this:


{% highlight js %}
var Order = function(params) {
  this.params = params;
};

Order.prototype = {
  save: function() {
    // save order
    console.log("Order saved");
    this.sendEmail();
  },

  sendEmail: function() {
    var mailer = new Mailer();
    mailer.sendPurchaseEmail(this.params);
  }
};

var Mailer = function() {};

Mailer.prototype = {
  sendPurchaseEmail: function(params) {
    console.log("Email sent to " + params.userEmail);
  }
};

> var order = new Order({ userEmail: 'john@gmail.com'  })
> order.save();
> "Order saved"
> "Email sent to john@gmail.com"
{% endhighlight %}

You got the idea, right? This code has a few problems. Probably the most
important one is that `Order` and `Mailer` are tightly coupled. Usually
you know two components are coupled when a change to one requires a change in
the other one, and that's the case. If we want to change the name of the
`sendPurchaseEmail` method or their params, we'll have to change `Order`
implementation as well. This change may look something pretty straightforward,
but in a large alication this bad practice means having a tightly coupled
alication where a small change can easily end up in a waterfall of changes.

Another problem you'll find using that approach is that you'll have to mock lot
of stuff in your tests, becoming a nightmare in large applications.

So now that we know the problems of this aroach, let's have a look at how we
can improve this code using the **Publish/Subscribe** pattern.

## First things first: what's the Publish/Subscribe pattern?
The [publish-subscribe pattern](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern), also known as pub/sub pattern, is defined as follows:

_In software architecture, publish–subscribe is a messaging pattern where
senders of messages, called publishers, do not program the messages to be sent
directly to specific receivers, called subscribers. Instead, published messages
are characterized into classes, without knowledge of what, if any, subscribers
there may be. Similarly, subscribers express interest in one or more classes,
and only receive messages that are of interest, without knowledge of what, if
any, publishers there are._

This pattern uses an event system which sits between the objects wishing to
receive notifications (subscribers) and the object firing the event (the
publisher). This event system allows code to define alication specific events
which can pass arguments containing values needed by the subscriber. The goal
is to avoid dependencies between the subscriber and the publisher.

## Using the pub/sub pattern in plain JavaScript applications
Now that we know how the pub/sub pattern works, we can use it to improve the
code shown above. Let's start creating out event system.

{% highlight js %}
var EventBus = {
  topics: {},

  subscribe: function(topic, listener) {
    // create the topic if not yet created
    if(!this.topics[topic]) this.topics[topic] = [];

    // add the listener
    this.topics[topic].push(listener);
  },

  publish: function(topic, data) {
    // return if the topic doesn't exist, or there are no listeners
    if(!this.topics[topic] || !this.topics[topic].length) return;

    // send the event to all listeners
    this.topics[topic].forEach(function(listener) {
      listener(data || {});
    });
  }
};

{% endhighlight %}

This is a simple implementation of the pub/sub pattern, you can find some
libraries that make a full implementation of the pattern like
[Radio.js](http://radio.uxder.com/),
[Amplify.js](http://amplifyjs.com/api/pubsub/) and many others.

Now we can use our brand new `EventBus` object to send messages between objects

{% highlight js %}
> EventBus.subscribe('foo', alert);
> EventBus.publish('foo', 'Hello World!');
{% endhighlight %}

Now let's see how to use the `EventBus` to decouple `Order` and `Mailer`

{% highlight js %}
var Mailer = function() {
  EventBus.subscribe('order/new', this.sendPurchaseEmail);
};

Mailer.prototype = {
  sendPurchaseEmail: function(userEmail) {
    console.log("Sent email to " + userEmail);
  }
};

var Order = function(params) {
  this.params = params;
};

Order.prototype = {
  saveOrder: function() {
    EventBus.publish('order/new', this.params.userEmail);
  }
};

> var mailer = new Mailer();
> var order = new Order({userEmail: 'john@gmail.com'});
> order.saveOrder();
> "Sent email to john@gmail.com"

{% endhighlight %}

You can see how `Order` and `Mailer` don't know anything about each other. Now
we can change the implementation of both functions without care about the
impact of the changes, it's just a matter of preserving the events we're firing
or listening to. Furthermore we can test both functions in isolation without
the need of mock objects, which is always a great thing.

## Using the pub/sub pattern in Backbone applications
Backbone provides a way to send, and listen to events using the
`Backbone.Events` module, that makes the pub/sub implementation pretty
straightforward, we just have to mix `Backbone.Events` into an empty object
like so:

{% highlight js %}
var EventBus = _.extend({},Backbone.Events);
{% endhighlight %}

Then we just use the standard `trigger` and `on` methods to publish and
subscribe to messages. Let's see the whole implementation:

{% highlight js %}
var EventBus = _.extend({}, Backbone.Events);

Foo = Backbone.Model.extend({
  sayHello: function(){
    var data = "Hello Wordl!"
    EventBus.trigger('event', data);
  }
});

Bar = Backbone.Model.extend({
  initialize: function(){
    EventBus.on('event', this.callbackMethod(data));
  };

  callbackMethod: function(data) {
    console.log(data);
  };
});
{% endhighlight %}

Pretty much the same as the example above for plain JavaScript applications
except here instead of using `publish` and `subscribe`, we are using `trigger`
and `on`.

## Conclusion
At HouseTrip we make an extensive use of this pattern. Our main app is a quite
big Rails app using Backbone in the front-end. Our Backbone views and models
generate lots of events, and these events sometimes involve more events. Using
the pub/sub pattern prevent us ending up with a tightly coupled system which
will be impossible to maintain and test.

It doesn't matter the language or the framework, tightly coupled systems is one
of the most common problems in modern web applications. We, as developers,
should be able to create modular and reusable code and the Publish/Subscribe
pattern plays an important role in creating better applications, loosely coupled,
flexible, scalable and easy to test.
