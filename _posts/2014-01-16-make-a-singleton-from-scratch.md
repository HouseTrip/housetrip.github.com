---
layout: post
title: How to create your own Singleton from Scratch
published: false
author: David Silva
author_role: Developer
author_url: https://github.com/davidslv
author_avatar: http://www.gravatar.com/avatar/6aec36daee2fcb518971daa7f2e0f544.png
summary: |
  Ruby provides you with lots of modules ready to use, one of them is the Singleton Module. Although it's a good thing to know what's under the hood and create your own from scratch.
---

One of the 23 described patterns by the [Gang of Four](http://en.wikipedia.org/wiki/Design_Patterns) is the [Singleton Pattern](http://en.wikipedia.org/wiki/Singleton_pattern), which says only one object of a particular class is ever created and all further references to objects of the singleton class refer to the same underlying instance.

As you may know, Ruby provides you a [singleton module](http://ruby-doc.org/stdlib-2.1.0/libdoc/singleton/rdoc/Singleton.html) that you can include in your class, and magically your class is a singleton out of the box.

But let's say for the sake of learning you want to create your own Singleton from scratch.

All the work done by the Ruby Singleton module is:

- creates the class variable
- initializes it with the singleton instance
- creates the class level `instance` method
- makes the `new` method private
- prevents your class to be a Eager class


{% highlight ruby %}
class MySingleton

  # This would cause a eager instantiation,
  # something that we want to avoid
  # @@instance = MySingleton.new

  def self.instance
    # this will prevent you to have a eager class
    @@instance ||= new
  end

  # and finally the `new` method becomes private
  private_class_method :new
end
{% endhighlight %}


An equivalent implementation of `MySingleton` would be, using the Ruby library:

{% highlight ruby %}
require 'singleton'

class RubySingleton
  include Singleton
end
{% endhighlight %}

We can demonstrate this by running both implementation through the same test suite

{% highlight ruby %}
require 'spec_helper'
require 'singleton/my_singleton'
require 'singleton/ruby_singleton'

shared_examples "a singleton" do
  it 'should not be a eager class' do
    # will return 1 if is a eager class and 0 if not
    ObjectSpace.each_object(described_class){}.should == 0
  end

  it 'should raise an error when calling the new method' do
    # `new` method is a private method
    expect {
      described_class.new
    }.to raise_error
  end

  it 'should always output the same instance' do
    singleton_instance = described_class.instance
    described_class.instance.should eql(singleton_instance)
    described_class.instance.class.should eql(singleton_instance.class)
  end
end

describe MySingleton do
  it_behaves_like "a singleton"
end

describe RubySingleton do
  it_behaves_like "a singleton"
end
{% endhighlight %}

You can learn more about [ObjectSpace](http://www.ruby-doc.org/core-2.1.0/ObjectSpace.html).

Happy coding!
