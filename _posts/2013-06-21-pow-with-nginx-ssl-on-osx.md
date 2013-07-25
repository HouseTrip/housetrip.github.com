---
layout: post
title: Configuring Pow with NGINX and SSL on OSX
published: true
author: Matt
author_role: Senior Developer
author_url: https://plus.google.com/107981766773306423214?rel=author
author_avatar: http://www.gravatar.com/avatar/9e0e76de0fe26c1326da1a232d4dd2f2?s=36
summary: This is a step by step guide on how to setup your local development environment to serve a Rails (or any Rack) app with [Pow](http://pow.cx/) and [NGINX](http://nginx.org) over HTTPS.  To begin I'm going to assume you're using OSX (probably Mountain Lion), [HomeBrew](http://mxcl.github.io/homebrew/) and [rbenv](https://github.com/sstephenson/rbenv).  For other setups ymmv.
---

This is a step by step guide on how to setup your local development environment to serve a Rails (or any Rack) app with [Pow](http://pow.cx/) and [NGINX](http://nginx.org) over HTTPS.

To begin I'm going to assume you're using OSX (probably Mountain Lion), [HomeBrew](http://mxcl.github.io/homebrew/) and [rbenv](https://github.com/sstephenson/rbenv).  For other setups ymmv.

### What is Pow?

> Pow (a [37signals](http://37signals.com) project) runs as your user on an unprivileged port, and includes both an HTTP and a DNS server. The installation process sets up a firewall rule to forward incoming requests on port 80 to Pow. It also sets up a system hook so that all DNS queries for a special top-level domain (.dev) resolve to your local machine.

[![Pow - artwork by Jamie Dihiansan](/images/2013-06-21/pow.png)](http://pow.cx)

For more information on Pow, read the [intro](http://pow.cx) or [browse the manual](http://pow.cx/manual.html).

### Why use Pow?

* Easily host multiple Rack apps on your local machine under different domains e.g http://your-app.dev
* Configure local apps to run under SSL (explained below)
* Use the [xip.io](http://pow.cx/manual.html#section_2.1.5) domain to visit your app from other devices on your local network
* Serve requests with multiple Pow workers
* Easy to configure, customise and works with multiple Rubies (via [rbenv](https://github.com/sstephenson/rbenv) or [RVM](http://pow.cx/manual.html#section_2.3.2)) and [Bundler](http://gembundler.com/)

### Installing Pow

Install Pow with this command;

{% highlight bash %}
  curl get.pow.cx | sh
{% endhighlight %}

Next create a symlink in ~/.pow to your app's base directory like so;

{% highlight bash %}
  ln -s /full/path/to/your-app ~/.pow/your-app
{% endhighlight %}

This should be enough for you to see your app working at http://your-app.dev. The next steps assume you have this working.

### Installing and configuring NGINX

Install NGINX via brew;

{% highlight bash %}
  brew install nginx
{% endhighlight %}

By default brew will install and configure NGINX to listen on port 8080. We need to run it on port 443 (decrypting SSL and proxy-ing all requests through to our Pow server).

Using [this config file](https://gist.github.com/matthutchinson/5815393) we can set up NGINX with some good defaults, and tell it to look for sites in `/usr/local/etc/nginx/sites-enabled`.

{% highlight bash %}
  mkdir -p /usr/local/etc/nginx/sites-enabled
  mkdir -p /usr/local/etc/nginx/sites-available

  curl -0 https://gist.github.com/matthutchinson/5815393/raw/9845b99433a0e1ebd2763b264643fe308ea74b4f/nginx.conf > /usr/local/etc/nginx/nginx.conf
{% endhighlight %}

Next we create our [site configuration](https://gist.github.com/matthutchinson/5822750) in `/usr/local/etc/nginx/sites-available`

{% highlight bash %}
  curl -0 https://gist.github.com/matthutchinson/5822750/raw/4790d7030d55a955b3c3a90fe2669b81235b95d2/your-app.dev > /usr/local/etc/nginx/sites-available/your-app.dev
{% endhighlight %}

Edit this file, setting the root (public) directory and replacing `your-app.dev` throughout. Finally symlink it into sites-enabled;

{% highlight bash %}
  ln -s /usr/local/etc/nginx/sites-available/your-app.dev /usr/local/etc/nginx/sites-enabled/your-app.dev
{% endhighlight %}


### Generating an SSL Cert

You might have noticed that the config file you just edited referenced an SSL cert that we have not yet created.

In a tmp directory, let's use [this handy gist](https://gist.github.com/matthutchinson/5815498) to generate it and move the cert files into place;

{% highlight bash %}
  curl https://gist.github.com/matthutchinson/5815498/raw/9da28acd6bf0ce1666f39cc0351dd5eee764be8b/nginx_gen_cert.rb > /tmp/nginx_gen_cert.rb
  ruby /tmp/nginx_gen_cert.rb your-app.dev
  rm /tmp/nginx_gen_cert.rb
{% endhighlight %}

You should now have SSL cert files for your app properly configured and contained in `/usr/local/etc/nginx/ssl`.

### Trying it out

Thats it! To start NGINX (since we are listing on port 443) you need to run it with sudo;

{% highlight bash %}
  sudo nginx
{% endhighlight %}

Visit https://your-app.dev/ now to see your app served via HTTPS.


### Controlling things

The web app can be restarted by running `touch tmp/restart.txt` in the base directory. And you can control NGINX from the command line with flags like this;

{% highlight bash %}
  sudo nginx -s start
  sudo nginx -s stop
  sudo nginx -s reload
{% endhighlight %}

### Debugging with pry-remote

Since your app is now running in Pow's own worker processes, to operate a live debugger you will need to use something like pry-remote.

First add the pry and pry-remote gems to your Gemfile (and `bundle install`). Then to introduce a breakpoint use this in your code;

{% highlight ruby %}
  binding.remote_pry
{% endhighlight %}

Fire off a request and when it stalls, run this command from your app's base directory;

{% highlight bash %}
  bundle exec pry-remote
{% endhighlight %}

A connection to the running worker process is established and you should be presented with a regular pry prompt.  You can read more about pry-remote and pry [here](https://github.com/mon-ouie/pry-remote).

### Further steps

Your browser may complain about not trusting your new SSL cert - we **can** fix that!

Restart or open Safari to visit https://your-app.dev and click 'Show Certificate' from the warning dialog.  Choose the 'Trust' drop-down and select 'Always Trust'. This adds your newly generated cert to the OSX keychain.

Setting up more sites is easy, just add them with a similar NGINX site config, generate an SSL cert (using the helper script again) and symlink things into place.

You can play with [Pow's configuration](http://pow.cx/manual.html#section_3) (e.g timeouts, workers) by defining ENV variables in ~/.powconfig, for example;

{% highlight bash %}
  export POW_DOMAINS=dev,test
  export POW_DST_PORT=80
  export POW_TIMEOUT=300
  export POW_WORKERS=3
{% endhighlight %}

Any change to ~/.powconfig needs a Pow restart;

{% highlight bash %}
  touch ~/.pow/restart.txt
{% endhighlight %}

I hope this guide has been useful. Comments or questions are always welcome.
