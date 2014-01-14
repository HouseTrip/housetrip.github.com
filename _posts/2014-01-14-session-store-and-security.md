---
layout: post
title: Session storage and security in Rails
published: true
author: Daniel Cruz
author_role: Developer
author_url: http://dncrht.github.io
author_avatar: http://www.gravatar.com/avatar/2759ab03ed5459d07d0a5e0075a675fa.png?s=36
summary: |
  We'll explore how Rails handles and stores the session information, what security implications it has, and what alternatives we can use.
---

One of the keys of Rails' success is that provides a full stack where you can
start from. This awesome full stack, as any other piece of engineering, was built upon design decisions.
You take them for granted, and when you code on Rails, you assume that decisions
others have taken for themselves are good for you too.

Precisely one of Rails' focuses is about letting you start easily a project just
a `rails new` command away. Now you have a nice directory structure, a database
connection, CSS/JS preprocessors… a cool new toy with batteries included. You delegate
to Rails the low level details, whilst you concentrate on the high level stuff that really matters to your project.

But, let's think for a second. Which of those things are you going to ditch
in the near future whose solely purpose was to let you start _quickly_?

Well, one of those is certainly the database connection. Nobody uses sqlite in high
concurrency web apps. No big deal, you change the database driver among a couple of other retouches and you are ready to go. Or not?

## Enter the sessions

Sessions is a way to store information about the current user, his context and his
browsing session, but keep it hidden to him. Another way to store information relevant
to the browsing session is using cookies. Obviously, nobody uses cookies to store
any important information, because cookies are clear text, public and easy to tamper
strings. If the user logs in to your app, and you store his user_id in a cookie,
you're asking for trouble. Hence, cookies are unsafe, and you must use sessions.

One of the things that came for free on `rails new`, is a default session store,
without any need to configure. And the nasty surprise is this is one of the things you assumed that they were good for you.

Rails cookie based session storage was introduced as the default in 2007, version 2.0.0.
Previously the default (in version 1.2.6) was a file store, but was ditched for
performance reasons (it's faster to avoid a disk access) and scalability reasons
(different servers may not share the same filesystem).

Then it seems that a plain cookie is storing your session variables after all.
Does it mean that session information can be modified as if we were storing our
information in a cookie? No, because sessions have a signature that prevents tampering.

## Accessing the session data

Unfortunately, breaking the anti-tamper signature is easy.

I've prepared 3 simple projects, using Rails 2, 3 and 4. All 3 have equivalent
functionality, controllers and routes. Uncompress the [file](/images/2013-11-24/sessions.zip)
and checkout branch `rails-2`, `rails-3` or `rails-4` to use one of the projects.

They have a `/users` resource done with a basic Rails scaffolding. It has been modified
to allow sending a special parameter to set a session value.

Let's start the dev server on the _rails-2_ branch and follow these steps in your browser:

 1. Visit `http://localhost:3000`. No session data has been set yet.
 2. Set session data adding the parameter `http://localhost:3000?option=gotcha`
 3. Visit `http://localhost:3000`. The header shows the session data you've just set.

<img src="/images/2013-11-24/set_session.png" class="center-image" alt="set session"/>
&#x20;<p class="caption">Setting a session value in the browser.</p>

Session data is just the value of the cookie named `_two_session`.

<img src="/images/2013-11-24/get_session.png" class="center-image" alt="get session"/>
&#x20;<p class="caption">Getting the session cookie with the browser.</p>

You can try to get the value of that cookie in any of the other projects: `_three_session` in _rails-3_ and `_four_session` in _rails-4_.

### Decode the session data

In Rails 2 & 3, the value of the cookie is a base64 encoded serialized string with
an added signature. Session data is thus almost clear text.

However, in Rails 4, the value of the cookie is an encrypted string. But if you
have access to the source code of the application you can use the built-in infrastructure to decode the session.

Get the session data in `session_data` by opening a Rails console and running the required commands:

<img src="/images/2013-11-24/decoding.png" class="center-image" alt="decoding"/>
&#x20;<p class="caption">How to decode the session cookie in different Rails versions.</p>

### Altering session data

With the session data in our hands, it's very simple to add our own values…

```
session_data['option'] = 'boom'
```

…regenerate the cookie…

<img src="/images/2013-11-24/encoding.png" class="center-image" alt="encoding"/>
&#x20;<p class="caption">How to regenerate the session cookie in different Rails versions.</p>

…to get the new payload:

`BAh7BzoPc2Vzc2lvbl9pZEkiJWZkODdhMWVjNWVlNzlkOWIzMzhkOTUzZGU1NDg0MzM5BjoGRVQ6C29wdGlvbkkiCWJvb20GOwZU--d572c860f3f1cea35dca07d52abb362f37412ba8`

And request the page with the new payload:

```
curl --cookie "_two_session=NEW_PAYLOAD" http://localhost:3000/users
```

Finally, behold the output. Boom!


## Session store alternatives

Rails provides several session store alternatives. All these store the session data
far from the user. The only link between the user and the session is the session identifier,
a very SHA1-like string. Although this session id can be tampered with using the same
techniques we've seen before, the chances that you guess a valid session id matching a current user are really low.

<img src="/images/2013-11-24/alternatives.png" class="center-image" alt="alternatives"/>
&#x20;<p class="caption">Configuration of several session store alternatives.</p>

In order to decide which session store is right for you, you might apply different
criteria regarding your knowledge of the technology it uses, costs of storage, reliability,
memory usage and others. But a good rule of thumb is to always have an eye on
performance as a criteria to include in your decisions.

To perform some benchmarks you can check out the Rails 4 project, already configured
with different backends in each branch `rails-4-memcached`, `rails-4-mysql`, `rails-4-redis` or `rails-4-sqlite`.

Start the development server and then run:

`ruby benchmark.rb`

…from another terminal. The first benchmark creates a session and reuses it several times.
The second one writes to the session each time.

<img src="/images/2013-11-24/charts.png" class="center-image" alt="charts"/>
&#x20;<p class="caption">Benchmarks results.</p>

Cookie-based is the fastest one. SQL-based storages are the slowest, MySQL being 1% slower
than SQLite (considering that here we only have have one user).
It is highly recommended to run these benchmarks in several environments, as it may render
different results depending on hard disk type (SSD or not), memory size and speed,
processor, configuration of the backend, load of the backend and even session size and
the kind of data being stored in the session.

## The bottom line

We've demonstrated that cookie-based sessions are not the safest way to preserve the session
data as they are publicly visible in Rails 2 & 3, and easily modifiable to anyone that has the `secret_token`.

Other problems with cookies are:

 - they can only hold up to 4 kb of serialized data. While it's a bad practice to store
 loads of data in the session, this can be a limiting factor in certain circumstances.
 - the cookie is always sent by the browser in same-domain requests, slowing requests as
 the cookie gets heavier. You can minimize this effect by serving your assets from a different domain.
 - sessions are invalidated in the client side as a result of the cookie expiring policy.
 This means that a session cookie will always be cosidered valid for the server, posing a
 security problem if a cookie falls on undesired hands. Alternative session stores are
 invalidated both server side and client side.

Session cookies are good enough for a quick project start, or a personal project.
But as your project starts to grow you should evaluate and choose an alternative according to your requirements.

Avoiding details will let you go far, but knowing them will let you go further. Try to
question your dogmas from time to time, because that will let you know your tools better.
