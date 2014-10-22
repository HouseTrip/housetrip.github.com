---
layout: post
published: true
title: Using machine learning to rank search results (part 1)
author: Julien Letessier
author_role: Software Engineer
author_url: http://github.com/mezis
author_avatar: http://www.gravatar.com/avatar/88683f31bdf05a8071fb08327b3919cb
---

A large catalog of products can be daunting for users. Providing a very fine
grained filtering of search results can be counter-productive: it leads them
from information overload to lack of choice. 

On e-commerce sites, this results in poor conversion---users leaving the site
without checking out.

The key is obviously to provide _relevance_ and _choice_, which is much more
complicated than it sounds, as different users may have very different tastes.

This describes how I explored a machine learning, neural networks based
solution to relevance ranking.

We'll demonstrate that an ANN (artificial neural networks) based approach can
provide better ranking than our historical ranking heuristics, and still provide
good performance.

Read the [complete article](http://dec0de.me/2014/10/learning-to-rank-1/) on my
blog.

Credits: Thanks to [Alfredo Motta](https://github.com/mottalrd) and [Andy
Shipman](https://github.com/mrship) for their helpful review of this article.

