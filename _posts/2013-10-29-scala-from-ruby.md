---
layout: post
title: Scala - first impressions of a Ruby developer
published: true
author: Karlo
author_role: Developer
author_url: https://github.com/Karlotcha
author_avatar: https://2.gravatar.com/avatar/5f0f07a8d1d189b183c1bde70cfdc595
summary: I am always curious about languages I don't know, even more so when a
 language is considered 'trendy'. But I am also careful and trendy doesn't mean
  good. So to make my own opinion, I decided to give Scala a try.

---

I am always curious about languages I don't know, even more so when a language
 is considered 'trendy'. But I am also careful and trendy doesn't mean good. So
 to make my own opinion, I decided to give Scala a try.

There are a lot of online ressources to learn it: I used mainly [coursera](https://www.coursera.org/course/progfun),
[tutorials from scala-lang](http://docs.scala-lang.org/tutorials/), [Scala for the Impatient](http://horstmann.com/scala/),
and the Scala console to play with it.

### Scala and Ruby, a lot of similarities\.\.\.

Both languages are **high-level** programming languages, multi-paradigm (**functional**
 and **object-oriented**), **strongly** typed. Both have a nice syntax, very expressive
 (IMHO). Scala and Ruby have become more alike with the latest releases of Ruby.
 Ruby 2.0 with Enumerator::Lazy, brought us lazy evaluation, a tool naturally present in Scala.
 It also added keyword arguments;

- In Ruby:
{% highlight ruby %}
  def cat(grumpy: 'mew', happy: 'mow'); end
  cat(grumpy: 'meh', happy: 'beh')
{% endhighlight %}

- In Scala:
{% highlight scala %}
  def cat(grumpy: String = "mew", happy: String = "mow") {}
  cat(grumpy = "meh", happy = "beh")
{% endhighlight %}

It is well known that in creating Ruby, Matz was partly inspired by existing ideas in other languages,
perhaps we should count Scala in that list now?

### ... but also some small differences \.\.\.

Among other things, I really like Scala's console. Like the Ruby console, this is a
 precious tool to play with the language and something that Scala betters Ruby
 on with **named** results;

{% highlight scala %}
    scala> 1+1
    res0: Int = 2
{% endhighlight %}

Do you see the **`res0`**? This is just the name of your result because everything your console
returns to you has a name! Which means if you need it later, you can just call **`res0`**.
It is so handy and so simple, isn't it?
Scala also has great docs and plenty of great online resources to learn more, the community too
seems quite active.

There are also a few minor differences between Scala and Ruby that disturb me.

You have different types for characters (that you declare with single quotes) and
strings (double quotes). This is a minor difference (or maybe not?) but when you come from Ruby
and are used to declaring your strings with single quotes, it is a bit surprising. Scala also has
 plenty of nice features for syntactic sugar like procedures (functions without returned values),
 singleton and companion objects (to manage static methods) and constructors. The list is quite
 impressive and this brief article is too short to go into every detail of the language.

### ... and huge differences \.\.\.

Besides that, Scala has some bigger fundamental differences from Ruby. First of all, it is **statically
typed**.  This can be good since you gain all the advantages of a statically typed
language (safeness and speed). Bad because you have to specify variable types which
can be a bit annoying. Scala does have a nice feature called **type inference** which helps it
guess types in some cases (mostly for returned values of expressions and functions as far as I know).

Scala is more functionally-oriented than Ruby. In particular, you have this static typing system,
immutability (which is much more prominent in Scala than in Ruby and its "frozen"), lazy evaluation,
pattern matching and currying. Ruby can offer similar features but they are more advanced and feel more natural
in Scala. There may be also a cultural difference here, I believe the Scala community is generally more enthusiastic
about functional programming (this point of view may be quite subjective though).

Another important aspect of Scala is its interoperability with **java**. Because Scala runs on the Java
Virtual Machine, it is possible to combine it with Java libraries quite naturally (which doesn't mean it
 is always easy). Because of the maturity of java, this could be considered a big advantage.
 However this feature has its downsides: you may have dependencies with java libraries when
  you use a Scala tool. And if you dislike Java (for any reason, good or bad), you should probably keep that in mind.

### Conclusion
I really loved my short experience with Scala. I think this language is very rich and I really wish to
play a bit more with it. I even believe I may become a better Ruby developer by doing so. There are
a few more things I wish to explore, like **play2**, a web framework in Scala comparable to Rails
or [Twitter's Scala Documentation](http://twitter.github.io/scala_school/), hopefuly a great source of inspiration!
