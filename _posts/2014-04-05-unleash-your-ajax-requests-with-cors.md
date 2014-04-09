---
layout: post
title: Unleash your AJAX requests with CORS
published: false
author: Daniel Cruz
author_role: Software Engineer
author_url: http://dncrht.github.io
author_avatar: http://www.gravatar.com/avatar/2759ab03ed5459d07d0a5e0075a675fa.png?s=36
summary: | 
 Cross-origin resource sharing is a mechanism that allows your AJAX requests to reach new territories. Let's refresh our knowledge on AJAX requests, their limits and how CORS helps to circumvent them.
---

Cross-origin resource sharing, or CORS, is a mechanism that allows AJAX requests to circumvent
their _same origin_ limits.

For demonstration purposes, we'll use a small Ruby project called _F1 race results_.
It presents a page with the results of the current F1 Grand Prix in _real time_. The user clicks
on a button to refresh the race standings while the page is kept on screen.

Download the [project](/images/2014-04-05/f1-race-results.zip), `bundle install` and launch it
with `foreman start`.

_**NOTE:** Please use the Firefox browser to test the exercises._

## A bit of history

The ability of a browser to request a resource from a server without reloading the page hasn't
been available since day one.

Back in the year 2000, websites that needed to do any kind of background request used alternative
techniques. One of these primitive techniques involved the use of an iframe.
Iframes, as JavaScript, were available in major browsers since 1996.

If we point our browser to [http://localhost:3000/iframe](http://localhost:3000/iframe) we'll see a
list of results. As intended, clicking in the button makes the page refresh the list of results but
avoiding a full page reload.

How is it done? The JS code in the page detects the click in the button and changes the `src`
attribute of a hidden `iframe` tag. This `src` change is detected by the browser, which will
request the resource asynchronously. The requested resource is in fact a small page that contains
a little JS snippet responsible of changing the race results in the main page.

<img src="/images/2014-04-05/iframe.png" alt="iframe flow" width="339"/>
&#x20;<p class="caption">Poor man's AJAX.</p>

The drawbacks with this approach is that you can't make POST requests and looks very hacky.

## Asynchronous JavaScript and XML

To avoid this hackyness, in 1999 Microsoft added to Internet Explorer 5 a JavaScript extension,
XMLHttpRequest, to make programmer's life easier. With XMLHttpRequest there's no need to use an `iframe` to do background requests.

It remained quite as a privative rarity until the early 2000s, when several webapps started using
it.

Shortly afterwards, somehow the technique was named as AJAX.
The name is rather misleading, as it allows not only the page to request a resource
asynchronously, but synchronously too. Even more, it's not restricted to XML, because JSON or
plain HTML can be sent as well. In those cases, the technique is often called
[AJAJ](https://en.wikipedia.org/wiki/AJAJ) and [AHAH](http://microformats.org/wiki/rest/ahah)
respectively. However, nobody is so specific, and the name AJAX is widely accepted.

The XMLHttpRequest is conveniently wrapped on jQuery by `.load`, `.get` and `.post` methods.

Back to our example app, in [http://localhost:3000/ajax](http://localhost:3000/ajax) clicking
the button will reload the results. Exactly like the previous example, just using this _new_
technique.

<img src="/images/2014-04-05/ajax.png" alt="AJAX flow" width="339"/>
&#x20;<p class="caption">Simplicity is the ultimate sophistication.</p>

However, AJAX is an open door for insecurity. Your page may contain a third party script that
silently sends your [session cookie](http://dev.housetrip.com/2014/01/14/session-store-and-security/) to another server.

### Same origin policy

To prevent security risks, the browser enforces a [same origin policy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Same_origin_policy_for_JavaScript).

What is considered as different [origin](https://tools.ietf.org/html/rfc6454)?

- when the domain is different
- when the protocol is different
- when the port is different

To see this policy in action, let's edit the file `app/views/ajax/index.html.erb` and replace the line:

{% highlight javascript %}
$('section').load('/cors/results');
{% endhighlight %}

by

{% highlight javascript %}
$('section').load('https://localhost:3001/cors/results');
{% endhighlight %}

and reload the page. Now clicking in the button won't refresh the standings.
The browser's network/console panes will be showing an error.
Why? It's preventing the request because the origin is different: although the domain remains unmodified, both protocol and port have changed.

## JSONP

A possible workaround for this is JSON with padding, known as [JSONP](http://www.json-p.org/).

It's a similar technique as the iframe's, but using a script tag. script tags are not bound to same origin as iframe tags are.

To see this in action, go to [http://localhost/jsonp](http://localhost/jsonp). After the
button is clicked, we create a script tag with the src attribute pointing to our resource.
The browser loads the resource, which contains pure JavaScript code.
This code calls a callback function already present in the original page.

<img src="/images/2014-04-05/jsonp.png" alt="JSONP flow" width="407"/>
&#x20;<p class="caption">JSONP in action.</p>

For convenience purposes, the script tag creation is abstracted by jQuery's `.ajax` call,
and we might want to replace our code by:

{% highlight javascript %}
$.ajax({
  url: 'https://localhost:3001/jsonp/results?callback=refreshSection',
  jsonp: 'refreshSection',
  dataType: "jsonp",
});
{% endhighlight %}

Nonetheless, this looks as hacky as the iframe technique, and doesn't support any other HTTP
method than GET. It can't be set synchronous as AJAX can, and it forces us to program the callback
function. Furthermore, it may open a big security risk as third party servers might include insecure or untrusted code.

What is the differences between this and 1996's `iframe` technique then? Simply put, an iframe can't modify the code of the parent page if it comes from a different domain, but a script can call the parent page.

## CORS

Finally, to overcome all these difficulties, the [Cross-origin resource sharing](http://www.w3.org/TR/cors/) specification was born. It's still a working draft, but widely accepted.

CORS-aware browsers, instead of preventing the cross origin AJAX straight away, send an
`Origin: domain.tld` header to the server. In our case this header looks like:
`Origin: http://localhost:3000`

The browser will see if server's response has an `Access-Control-Allow-Origin: domain.tld`
header. If so, and the domain specified in the header matches the domain that originated the
request, the response will be accepted. In our case, the response header is:
`Access-Control-Allow-Origin: http://localhost:3000`

The header can be easily set in a Rails controller with just:

{% highlight ruby %}
headers['Access-Control-Allow-Origin'] = 'http://localhost:3000'
{% endhighlight %}

You can see all this in action going to [http://localhost/cors](http://localhost/cors). Show
the network pane in your browser and click the refresh button in the page.

<img src="/images/2014-04-05/cors.png" alt="CORS flow" width="339"/>
&#x20;<p class="caption">Almost like the AJAX one with few extra headers.</p>

Unlike previous examples, in this case we are performing a cross origin POST request, instead
of GET. How cool is that?

### Fine grain control

The theory behind the headers is initially simple. But the server's response can be tuned for
certain requests. If we want to restrict GET requests, the server should have this header:
`Access-Control-Allow-Method: POST`

Check the whole list on [http://www.w3.org/TR/access-control/#resource-requests](http://www.w3.org/TR/access-control/#resource-requests).

### Preflight response

CORS requests are sent straight to the server, unless:

- HTTP method is not _simple_, i.e. other than: `GET`, `POST` or `HEAD`
- Content-Type is not _simple_, i.e. other than: `application/x-www-form-urlencoded`, `multipart/form-data` or `text/plain`
- request has [authentication headers](http://www.nczonline.net/blog/2010/05/25/cross-domain-ajax-with-cross-origin-resource-sharing/)

…among others. Check the full [list of conditions](http://www.w3.org/TR/cors/#resource-preflight-requests).

In any of these scenarios, the browser will do first a _preflight request_. This is simply
a request using the `OPTIONS` HTTP verb. If the request succeedes, the browser will issue the actual request right afterwards.
This preflight request is cached by the browser so the server is not bothered more than
necessary.

A very common case is a `POST application/json` request. It is very advisable to test if your
server is accepting preflight requests from the command line.

{% highlight bash %}
curl \
-k \
--verbose \
--request OPTIONS \
https://localhost:3001/cors/results \
--header 'Origin: http://localhost:3000' \
--header 'Access-Control-Request-Headers: Origin, Accept, Content-Type' \
--header 'Access-Control-Request-Method: POST'
{% endhighlight %}

#### Include authentication

If our server requires authentication, the AJAX/CORS call has to propagate the credentials.
Basic auth credentials are part of the request headers and will fall under the cases when the browser has to do a preflight request.

Simply add the `xhrFields` parameter to include auth credentials. Example:

{% highlight javascript %}
$.ajax({
    type: 'GET',
    url: 'https://domain.tld/path/to/resource',
    data: null,
    dataType: 'text',
    success: function(response) {

    console.log(response);

  },
  xhrFields: {
    withCredentials: true
  },
  error: function(error) {

    console.log('ERROR:', error);

  }
});
{% endhighlight %}

### CORS and Rails

Setting all these headers can be cumbersome, if we have to do it manually in every controller
or with a before_filter. It's even worse if our requests need a preflight response, because we
would have to set up `routes.rb` too.
Wouldn't be nice to have a middleware that does the heavy lifting for us? Granted: [Rack::Cors](https://github.com/cyu/rack-cors).

Have a look at `config/application.rb`:

{% highlight ruby %}
config.middleware.use Rack::Cors do
  allow do
    origins 'localhost:3000'
    resource '*', headers: :any, methods: %i(get post)
  end
end
{% endhighlight %}

Remove these block, restart the server and try again [http://localhost/cors](http://localhost/cors). It won't work.

#### Caveats

The string `'*'` cannot be used to define origins if we need to support credentials.

It is worth noting that you have to disable `protect_from_forgery` (CSRF protection) when
doing POST requests because we're dealing with different servers that doesn't share sessions.

### CORS and a cloud storage facility

A brief note if you store static resources in [S3](http://aws.amazon.com/es/s3/). You can
still specify the CORS headers in your bucket's Properties ➝ Permissions tab. Please refer
to the [official documentation](http://docs.aws.amazon.com/AmazonS3/latest/dev/cors.html).

There's documentation for [Google](https://developers.google.com/storage/docs/cross-origin?hl=es) as well.

## CORS current limitations

CORS is available in Chrome, Opera 12, Safari 4, Firefox 3.5 and Internet Explorer 10.
Support in mobile browsers vary, although works fine in latest iOS and Android.

### Chrome

Chrome works fine in the vast majority of scenarios.

You may encounter difficulties when the request is targeting a HTTPS server using
[self signed certificates](https://code.google.com/p/chromium/issues/detail?id=96007).
A possible workaround would be [add the certificate](http://stackoverflow.com/questions/15115746/why-chrome-cancel-cors-option-request)
in your trusted list. The certificate that comes with the `thin` webserver will fall under this category.

Another problem would be if you use basic auth and HTTPS. Chrome needs you do a manual
request to any resource protected by HTTPS and basic auth to make the CORS request work.

### Internet Explorer: a classic contender in any "limitations" section

IE 8 an 9 have some form or CORS support, but [severely limited](http://blogs.msdn.com/b/ieinternals/archive/2010/05/13/xdomainrequest-restrictions-limitations-and-workarounds.aspx).
The support is provided by a new JavaScript object, XDomainRequest, unable to:

- do any other request than GET or POST
- add authentication headers
- request nothing but `text/plain`
- target a different protocol

It's better to stay away of CORS and IE 8/9. In these situations, you may want to implement
any of the techniques previously described, as a fallback.

IE 10 has [full support](http://blogs.msdn.com/b/ie/archive/2012/02/09/cors-for-xhr-in-ie10.aspx)
though, out of the box, via the standard XMLHttpRequest.

## Conclusion

We've depicted how to make background requests in several ways, from the fundamentals to the latest standards.

Here at [HouseTrip](http://www.housetrip.com) we consider the CORS technology is mature enough,
and we successfully use it for the sign in and sign up forms, which are sent securely to the server.

I really hope this exploration process has helped you to understand how AJAX/CORS work, and
reduce the trouble you'll probably have setting it up in your next project.
