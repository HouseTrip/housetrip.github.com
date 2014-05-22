---
layout: post
title: Page objects, or how to better test by wrapping HTML with code
published: true
author: Alessandro Mencarini
author_role: Software Engineer
author_url: https://plus.google.com/u/0/104356313878293638468?rel=author
author_avatar: http://www.gravatar.com/avatar/d7015e59531008a4e43ff574400d5d87.png
summary: |
  Using page objects can help greatly with testing DOM or portions of it.
---

A typical RSpec feature (and Cucumber steps won't differ too much) could present this sort of code:

{% highlight ruby %}
describe "property page" do
  let(:price) { 90 }
  let(:property) { Property.create(price_per_night: price) }

  it "shows the price per night of the property" do
    visit property_path(property)
    expect(page).to have_content('From €90 per night')
  end
end
{% endhighlight %}

If this were the only test visiting the property page, you'd be ok. But you'll probably want to do this in more than one test.

What if you end up changing the route, making all `visit` calls break?
What if you want to add a query parameter to all calls that go to your property page?
What if a CSS class is changed and all tests working with it now fail?

You can find and replace all those instances in your test suite.
If you go that way, be sure you have a cup of camomile tea at hand, as the process is bound to be unnerving.

Or...

Enter [page objects](http://martinfowler.com/bliki/PageObject.html).

{% highlight ruby %}
module Pages
  class Property
    include Capybara::DSL
  end
end
{% endhighlight %}

Adding a layer that gives you a clean Ruby interface to interacting with the UI will make your life easier. Let's see how to refactor the tests involving our property page into a page object.

{% highlight ruby %}
def open(property)
  visit property_path(property)
end
{% endhighlight %}

Rather than keep using the `visit(path)` syntax directly in our tests, we have a central spot of our code to refer to, whenever we want our test driver (i.e. Capybara + Poltergeist, in our case) to go to that page.

Let's say that there is a new requirement, and all calls to property pages must include the `?background=blue` query string. Easy as pie:

{% highlight ruby %}
def open(property)
  visit property_path(property, background: 'blue')
end
{% endhighlight %}

Just one change. Now, let's refactor the contents check.

{% highlight ruby %}
def has_price?(price)
  page.has_content?("From #{price} per night")
end
{% endhighlight %}

After an AB test it's shown that a more frisky way to show the price converts better. Let's edit the test to mirror the changes in the page:

{% highlight ruby %}
def has_price?(price)
  page.has_content?("It's only #{price} for a night, yo!")
end
{% endhighlight %}

Remember when you had to change this a hundred times in your test suite? No more. The RSpec feature would look somewhat like this:

{% highlight ruby %}
describe "property pages" do
  let(:current_page) { Pages::Property.new }
  let(:price) { 90 }
  let(:property) { Property.create(price_per_night: price) }

  it 'should show the price per night'
    current_page.open(property)
    expect(current_page).to have_price(price)
  end
end
{% endhighlight %}

For a more thorough implementation of this concept you may also check [SitePrism](https://github.com/natritmeyer/site_prism).

