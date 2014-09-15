---
layout: post
title: Writing Specs without Rails for Legacy apps
published: true
author: Kris Leech
author_role: Software Engineer
author_url: http://teamcoding.com
author_avatar: http://www.gravatar.com/avatar/5923e70879e9586f2bf1c301e3f80e22.png
summary: |
    The why and how of writing specs without requiring Rails for legacy apps in order to move towards a hexagonal style architecture.
---

In our MonoRail we have a _lot_ of specs, the majority of which `require 'spec_helper'`, which in turn requires Rails. This _is_ necessary for any specs which depend on the Rails framework like controller and acceptance tests.

But for many specs, such as unit test, where we test one object in isolation and intergration tests, where we test the interactions between multiple collaborating objects, there is no need to boot the entire Rails framework.

Apart from Rails being slow to boot the main advantage of not requiring it is that none of your application code is automatically required.

This means we have to manually require each dependency of the code we are testing. This in turn makes us more aware of the dependencies and relationships between our units of code.

The idea being that we want to decouple our core business logic, such as models and services, from the delivery mechanism (HTTP/HTML). This is the central idea behind [Hexagonal Rails](http://rubyrogues.com/078-rr-hexagonal-rails-with-matt-wynne-and-kevin-rutherford/) (aka [Port and Adapters](http://alistair.cockburn.us/Hexagonal+architecture)). By having a clear boundary between core business logic and the delivery mechanism(s) the core application can be wrapped in a Web UI, JSON API, native app, Asynchronous process or CLI.

A good first step and the new default for the [rspec-rails](https://github.com/rspec/rspec-rails) gem is to have a separate `spec_helper` and `rails_helper`. The `spec_helper` file will _not_ require Rails, the `rails_helper` will require Rails. This makes requiring of Rails and its auto loading magic opt-in.

In a legacy app this will move awareness of dependencies to the forefront and present opportunities for refactoring towards a Hexagonal architecture.

## How

The first step is to rename your existing `spec_helper` to `rails_helper` and find/replace all instances of `spec_helper` with `rails_helper`. Your specs should still pass. Next create a new `spec_helper` file, copying across all, if any, none-Rails related stuff. Minimal is good, the closer to the following example the better:

{% highlight ruby %}
# do not add anything here unless it is required by EVERY spec

RSpec.configure do |config|
end
{% endhighlight %}

Finally add `require 'spec_helper'` at the top of `rails_helper`.

Once merged let your team know:

* `spec_helper` no longer boots Rails
* If you need the entire Rails framework require `rails_helper`

A pleasant side effect is that tests which don't boot Rails run faster giving a shorter feedback cycle when doing TDD.

The default for anything which is intrinsically dependent on Rails (e.g. controllers/helpers/views/routing) should use `require 'rails_helper'`.

Anything else; models, presenters, services etc. should use `require 'spec_helper'`.

### Be proactive

If you touch a spec that requires `rails_helper` see if you can remove it and just use `spec_helper`.

## Tips

### Rails constant

By not booting Rails it means any references to the `Rails` constant such as `Rails.env` will break. Instead you can pass the environment in to the object from the outside.

### require and LOAD_PATH

In specs without Rails none of the app directories are added to your `LOAD_PATH` which means a simple `require 'user'` will not work. Instead you can use `require_relative`, but this tends to ends up being ugly `require_relative ../../../app/models/user`.

Instead you can use the [spec_requirer](https://github.com/HouseTrip/spec_requirer) gem which adds methods which under the hood do a `require_relative`:

{% highlight ruby %}
require_models     'user', 'property'
require_presenters 'user_presenter'
{% endhighlight %}

Here is an example configuration which you could add to `spec_helper`:

{% highlight ruby %}
require 'spec_requirer'

APP_ROOT = Pathname(File.dirname(__FILE__)).join('..')

Spec::Runner.configure do
  components = %w(models services forms jobs queries validations presenters)

  SpecRequirer.setup(app_root: APP_ROOT.join('app'), components: components)

  # spec_requirer does not (yet) have support for requiring outside the app directory, so we do these manually.
  def require_initializer(name)
    require APP_ROOT.join('config', 'initializers', name)
  end

  def require_support(name)
    require APP_ROOT.join('spec', 'support', name)
  end

  def require_lib(name)
    require APP_ROOT.join('lib', name)
  end
end
{% endhighlight %}

## Summary

We can better decouple our core business logic from our delivery mechanism if we become aware of our dependencies by explicitly requiring them.

The first step is for `spec_helper` to not boot Rails and its autoloading magic.
