---
layout: post
published: false
title: Regaining lost memories
author: Bence Monus
author_role: Software Engineer
author_avatar: http://www.gravatar.com/avatar/26563a0d8435bd966527978473d9b85b
summary: A few things you can try to keep your Rails app's memory usage in check.
---

Here at HouseTrip we have a couple of apps running on Heroku. We often have memory issues with one of them so I took some time experimenting a bit with different combinations of the stack and various small tweaks here and there to see if I can pull memory usage down a bit.

A 1X dyno (a worker on Heroku) comes with 512MB memory which is not a whole lot but generally it should be enough. The app also picks up lots of memory usage over time.

The stack:

* Rails 4.0.3
* Ruby 2.1.1
* web server: Unicorn

## Preparation

### Enabling resource monitoring on Heroku
Heroku comes with resource monitoring which can be enabled using the control app.

`
$ heroku labs:enable log-runtime-metrics
`

It makes some useful log messages pop up periodically in `$ heroku logs` like this:

```
sample#memory_total=128.58MB sample#memory_rss=119.44MB sample#memory_cache=9.14MB sample#memory_swap=0.01MB sample#memory_pgpgin=0pages sample#memory_pgpgout=39023pages
```

### The benchmark
Please, do not take the numbers seriously. It is a **very** superficial experiment as far as benchmarks go. I used [siege](http://www.joedog.org/siege-home/) to hit the search controller using 10 threads. I let it run for a few minutes, check memory usage, let the app cool down a bit and check memory usage after a few minutes of inactivity again. 

## Initial performance

The app using Unicorn usually boots up with a ~120MB memory footprint which easily goes up to 250MB after processing a few requests. Under load it is above 300MB but it seems to consume more and more memory over time.

## The first experiment: JRuby

It was a semi-serious idea, JRuby is not famous for operating with a small memory footprint but I wanted to see if running ruby over JVM solves the issue of losing free memory over time.

Switching to JRuby was not without problems (as expected). I had to disable a few gems that come with native extensions, change **mysql2** to **jdbc-mysql**, etc. After a bit of disabling features and quick-fixing a few broken things I managed to get the app up and running for some load testing.

The app booted up with a whopping 250+ MB memory usage. It is a lot. Under load it went up to 450MB. After the load stopped it was around 250MB again, which is nice, at least the GC in JVM is doing something. But all in all the result wasn't very convincing. The response time was also significantly slower, although it was slowly gaining speed as the testing was going on.

## The second experiment: Puma

The second idea was to try Puma, this new and shiny web server. It is said to be very memory friendly so I gave it a go.

Note: I switched back to Ruby 2.1.1 for all the subsequent tests even though the Puma homepage says that Puma is designed to perform the best on a ruby implementation that supports proper threading such as JRuby or Rubinius.

The app booted up with a nice ~80MB memory usage which went up to 120-130MB after the load testing and seemed to stay there pretty constantly even after running the benchmark a few more times. Thumbs up.

## The third experiment: tweaking dependencies and Rack

### Rails libraries

Rails is loaded in `config/application.rb` usually by this line:

```
require 'rails/all'
```

Quite often the whole Rails universe is not required for an app so I changed it to

```
require 'active_record/railtie'
require 'action_controller/railtie'
```

The excluded libraries include ActionMailer among other things, which is not needed.

### Rack
I went through the middleware list which can be obtained by running

`
$ bundle exec rake middleware
`

It has lots of things that might not be needed, I ended up adding this code to `config/application.rb`:

```
config.middleware.delete(Rack::Lock)
config.middleware.delete(Rack::ETag)
config.middleware.delete(Rack::ConditionalGet)
config.middleware.delete(ActionDispatch::RequestId)
config.middleware.delete(ActionDispatch::RemoteIp)
config.middleware.delete(Rack::MethodOverride)
unless Rails.env.development?
  config.middleware.delete(ActionDispatch::Reloader)
end
```

### The results
It did not have an impact on memory consumption whatsoever. I did not expect a significant boost but a bit of an improvement would have been nice. Removing the middleware had an interesting side-effect, though. It sped up response time a **lot**. My benchmark showed an almost twofold increase in the average response time. Which is thumbs up again.

## Conclusion
I'm not going to compare Unicorn and Puma since I don't have any real data to back it up and it would be way out of the scope of this experiment. Also, I didn't care about performance, I was simply monitoring memory usage.

Puma seems to have a significantly smaller memory footprint. I can't really add anything else here. We have to see how it performs out there in the wilderness.

Micro-managing dependencies is not really worth the trouble. Although I feel better knowing that some unneeded libraries are not loaded, so there's that.

Pruning middleware can speed up the time it takes for Rack to process each request. Of course it is not going to make your database queries run faster but for short requests it might give you a measurable improvement. 

## Update

* Removing `ActionDispatch::Callbacks` seems to have some weird side-effects, experiment with it at your own risk. Related snippet updated.
