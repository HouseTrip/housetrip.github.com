---
layout: post
title: Testing Active Resource Models
published: true
author: Jesper Kjeldgaard
author_role: Developer
author_url: http://github.com/thejspr
author_avatar: http://www.gravatar.com/avatar/e96273c9bd8ea77a17ae87dca4c0de4c
summary: This post covers ways to test Active Resource models
---

We are currently in the process of moving towards a more service oriented
architecture, and have more specialized services instead of one big application.
One of the big advantages of SOA is that it becomes easier to add features and
deploy changes, given the code base is smaller and the service only manages one
part of the business domain. This is especially relevant for large teams (like
here at HouseTrip) where changes are added and deployed on an almost hourly
basis.

Moving to an SOA does not happen overnight, and it can be hard to fit in
refactoring's like this when there is a constant need to add and refine features
to a product. It is best achieved one step at a time, and one such step can be
to move an Active Record model to use Active Resource.


## Turning a model into an Active Resource

It's really easy to turn an Active Record model into an Active Resource. All
that's needed it to change the class the model inherits from and add the uri for
the resource endpoint.

{% highlight ruby %}
# before
class User < ActiveRecord::Base
end

# after
class User < ActiveResource::Base
  self.site "http://users.services.com/"
end

{% endhighlight %}

The next step is then to build an api for the resource to consume. Active
Resource works out of the box with RESTful endpoints similar to the ones
generated via `rails generate scaffold users`. The mapping is roughly as
follows:

{% highlight ruby %}
User.all      => GET'/users'
User.find(id) => GET '/users/:id'
user.save     => POST '/users/:id'
user.destroy  => DELETE '/users/:id'
{% endhighlight %}

You only need to implement the endpoints you use, e.g. if all you need is
`find`, then `/users/:id` will be the only endpoint you need.

## Testing an Active Resource

Moving your data into a separate service imposes some inconveniences on your
test suite, mainly it requires that your either have the service running during
tests or you mock out the responses to it. Running the service during tests is
quite inconvenient, especially during development. Hence, mocking the service
appears to be the best approach. Active Resource comes with some useful
functionality which makes it easy to mock specific requests, mainly
`ActiveResource::HttpMock` and `ActiveResource::Request`. The following snippet
shows how to test `User.all`.

{% highlight ruby %}
describe User, '.all' do
  context 'there are no users' do
    it "returns an empty array" do
      ActiveResource::HttpMock.respond_to do |mock|
        mock.get '/users', {'Accept' => 'application/json'}, []
      end

      User.all.should be_empty
    end
  end

  context 'there are 2 users' do
    it "returns all users" do
      ActiveResource::HttpMock.respond_to do |mock|
        mock.get '/users', {'Accept' => 'application/json'}, two_users
      end

      User.all.size.should == 2
    end
  end
end
{% endhighlight %}

If you want to test your architecture a bit more thoroughly, the [vcr](https://github.com/vcr/vcr) gem is a
great way to record responses from your service for reuse when running tests.

## Using a test store

If you're working on a large application, you probably depend on having models
present between test steps, and here it can become a bit inconvenient to have to
mock all requests to the service. A solution to this problem is to write a small
class which is used as an in-memory store for the resource.

{% highlight ruby %}
class User::TestStore

  cattr_accessor :users

  @@users = {}

  def self.find_or_create_by_email(email)
    find_by_email(email) || User.make(:email => email)
  end

  def self.find_by_email(email)
    @@users.fetch(email) { :not_a_user }
  end

  def self.find_by_email!(email)
    @@users.fetch(email)
  rescue IndexError
    raise "User '#{email}' not found in User::TestStore"
  end

  def self.all
    @@users.map { |_,v| v }.compact
  end

  def self.add(user)
    @@users[user.email] = user
    user
  end

  def self.reset
    @@users = {}
  end
end

Before do
  User::TestStore.reset
end if defined?(Before)
{% endhighlight %}

This class makes it convenient to find User models in later steps of a test as
well as including some convenient methods to create users and reset the
storage.

## What we learned

Whilst it was very painless to move the model out from the application, the real
value of moving a resource to a service is the next time it has to be changed.
It is usually a lot easier to make changes in a small specific application, than
a large complex one. It feels good to reduce the complexity and size of a large
application, consider it a prepaid pleasurable experience for the next developer
that has to work on the model.
