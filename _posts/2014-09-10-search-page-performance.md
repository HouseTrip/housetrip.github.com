---
layout: post
title: A journey through a Web App optimization session
published: false
author: Alfredo Motta
author_role: Software Engineer
author_url: http://www.alfredo.motta.name
author_avatar: http://www.gravatar.com/avatar/c1044117a60a9c37a232cf8b6e2c87a8.png
summary: |
    Fast web apps are beautiful to use and are one of the key requirements for a successfull customer experience. In this blog post I will show you how to analyze the performance of your web app and take actions to speed it up.
---

## Identify the bottlenecks
Fast web apps are beautiful to use and are one of the key requirements for a successfull customer experience. In this blog post I will show you how to analyze the performance of your web app and take actions to speed it up. 

The first step is to get clear evidence of which are the biggest performance bottlenecks. This is a key step for a successful optimization. If you know your bottlenecks and their corresponding impact you can built an optimization plan that will prioritize them. 

Recently at HouseTrip I was working on optimizing the response time of our [search page](housetrip.com/en/search-holiday-apartments/london). In the following I will share with you how I approached the problem, the tools I used, and finally the solutions that have been implemented.

My investigation usually starts from the browser. This is what your end user is relying on and this is where you can get an immediate feeling of what's wrong with your page. The Google Chrome developer tools can give you an impressive amount of information. In the [dev tools documentation](https://developer.chrome.com/devtools/docs/timeline) you can find all the informations you need to master them.  

The simplest way you can start your analysis is to open the dev tools and go to the network panel. Here you must turn the recording on, load your page, and after the page loaded you can order the requests by response time. At this point you should see something that looks like the following:

[![Front End Performance]({{ site.url }}/images/2014-09-10-search-page-performance-front-end_thumb.png)]({{ site.url }}/images/2014-09-10-search-page-performance-front-end.png)

No rocket science, but you get a clear picture of what's going on. In this case it's obvious that our backend is the biggest bottleneck within the set of requests that happen in our page. It's definitely a good idea to dig deeper and see why this is taking so long.

To analyze the performance of our backend we can use [New Relic](http://newrelic.com/) but in theory anything that is able to record the execution time of your backend code would work. The feature that is most helpful to debug our performance issue is the "transactions monitoring". Here I can search within my endpoints and have sample execution traces that I can analyze up to the function level. In the following picture you can see the same endpoint that we hitted from the browser and the corresponsind execution time. With no big surprises all the time is spent in the controller.

[![Backend Performance]({{ site.url }}/images/2014-09-10-search-page-performance-backend_thumb.png)]({{ site.url }}/images/2014-09-10-search-page-performance-backend.png)

Digging deeper into the execution trace we can find all the informations we need. Here I will focus on two elements that caught our eye and that are shown in the picture that follows. 

* The first one is that we are spending a lot of time in rendering the list of properties you see in the [HouseTrip search page](http://www.housetrip.com/en/search-holiday-apartments/london). 
* The second one is that we are issuing lots of extra queries to retrieve the default photo of each property. 

[![Backend Performance Details]({{ site.url }}/images/2014-09-10-search-page-performance-backend_details_thumb.png)]({{ site.url }}/images/2014-09-10-search-page-performance-backend_details.png)


### Speedup the page

Now that we know what are elements slowing down our page we can start thinking about how to fix them. Luckily for us solutions for the above mentioned problems are well known and easy to implement.

To speed up the template rendering we can use some simple caching mechanism. One of the templates that was easy to cache in our case was the filters template that looks as follows:

[![Filters]({{ site.url }}/images/2014-09-10-search-page-performance-filters_thumb.png)]({{ site.url }}/images/2014-09-10-search-page-performance-filters.png)

This is easy to cache because the information in the template is almost static. Each filter can be selected/not-selected, it can be grayed out, and it could be in a number of locales. Apart from these combinations the content of the filters is static and does not varies over time. The cache key for the template will look as follows:

{% highlight rb %}
Digest::SHA1.hexdigest([name, selection, I18n.locale, CACHE_GENERATION].join('|'))
{% endhighlight rb %}

where `name` is the filter column we are caching (price, apartment type, etc.), `selection` is the set of currently selected options for this filter column, `I18n.locale` is the locale, and finally `CACHE_GENERATION` is the generation of our cache which is useful if you want to expire the cache for any reason. 

Now we are left with the problems of the extra queries for each property to retrieve the photo. When we retrieve the set of properties in the search result page each property has a default photo associated. This information is stored in a separate table and if you simply run:

{% highlight rb %} 
p = Property.where(title: 'Great beach house')
p.default_photo
{% endhighlight rb %}

this is going to generate two queries. One to retrieve the property, and the second one to retrieve the photo. Luckily there is an easy solution to this problem. We can simply pre-load the default_photo information by using the active record `include` option ([see docs](http://api.rubyonrails.org/classes/ActiveRecord/QueryMethods.html#method-i-includes)) as follows:

{% highlight rb %}
p = Property.includes(:default_photo).where(title: 'Great beach house')
p.default_photo
{% endhighlight rb %}

Now active record is going to preload in memory the information related to the `default_photo` and you will not need an extra query when you issue the `p.default_photo` instruction.

These two very simple fixes gave us a `20%` improvement on the search page response time and we can get much more by further caching the properties templates. These represents another `40%-50%` of the response time but are not as static as the filters and therefore are less trivial to be cached. 

### Conclusions

Optimizing your web app is not a trivial job. It requires a lot of investigation before you can actually find what are the main performance bottlenecks affecting it. In this post I have shown the entire journey to identify and solve two of the most common performance bottlenecks in your web app: (i) Template rendering and (ii) inefficient SQL queries due to the object-relational mapping. To do that I have presented a real world example we worked on here at HouseTrip.

In general performance problems could be way more difficult than what I presented here. For example if your database is suffering of high load then you probably need to re-think about your system architecture, but for the majority of cases the solutions I presented here will be good for a vast majority of cases.

Hope you enjoyed the journey and that you are looking forward to jump back to your app and speed it up!
