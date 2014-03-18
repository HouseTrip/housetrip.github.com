---
layout: post
title: Nested Resources with Backbone
published: false
author: David Silva
author_role: Associate Software Engineer
author_url: https://github.com/davidslv
author_avatar: http://www.gravatar.com/avatar/6aec36daee2fcb518971daa7f2e0f544.png
summary: |
  I want to share my solution to a topic that seems wide spread throughout the Internet without using any third party plugin to achieve it on Backbone, and for that I will use a 3 level nested resource as an example, giving you the necessary insight to make you fearless in this subject.
---


The topics I wish to discuss are [Backbone and Rails Nested Routes](http://stackoverflow.com/questions/8332249/backbone-and-rails-nested-routes) and [How do I define nested resources for backbone.js?](http://stackoverflow.com/questions/6838241/how-do-i-define-nested-resources-for-backbone-js).

One other thing I wanted to avoid was introducing another plugin in our codebase such as [Backbone Relational](http://backbonerelational.org/).

#### For the sake of this article I will use `Articles`, `Comments` and `Comment Replies` as examples.

Consider the following url structure: `/articles/:article_id/comments/:comment_id/comment_replies`

This means one article can have several comments, and one article comment can have a several replies.

Consider the following json structure for a `comment`:

{% highlight javascript %}
{
  id: 1,
  body: 'Sunny day in London!',
  url_root: '/articles/1000/comments/1'
}
{% endhighlight %}

The `url_root` attribute is very important, since it's the attribute that will make our life's easier in the moment we need to reach the `comment_replies` url.

After you prepare your `json` response and assuming you have a collection, it all starts with setting a url in the collection, like the following:

{% highlight html %}
<h1>Article Page</h1>

<script type="text/javascript">
  // The url param is the one responsible to make
  // all the rest work, this is where everything begins
  var comments = new App.Collections.Comments({ url: '#{ article_comments_path(@article) }'});
</script>
{% endhighlight %}

In your `Comments Collection` make sure you set the `url` you passed to the collection,
this is accomplished in the `initialize` of the collection, by passing the url as an option.

You will need to have that endpoint if you are thinking about [CRUD](http://en.wikipedia.org/wiki/Create,_read,_update_and_delete) since Backbone uses that path to make a `POST` for example.


{% highlight javascript %}
App.Collections.Comments = Backbone.Collection.extend({
  model: App.Models.Comment,

  url: function() {
    // /articles/:article_id/comments
    return this.baseUrl;
  },

  initialize: function(models, options) {
    this.baseUrl = options.url;
  }
});
{% endhighlight %}


The `Comment Model` is the middle resource between the `Article` and the `Comment Reply`, **it's important** that one of the attributes is the url with the `article_id` and the comment `id`.


{% highlight javascript %}
App.Models.Comment = Backbone.Model.extend({
  url: function() {
    // /articles/:article_id/comments/1
    return this.get('url_root');
  },

  initialize: function(options) {
    // Here is where we manage to build the nested url
    new App.Models.CommentReply({
      commentPath: this.url()
    });
  }
});
{% endhighlight %}


Finally the `CommentReply Model` receives the url with the necessary attributes in the `initialize` and sets it in the url function, allowing it to interact with the server.


{% highlight javascript %}
App.Models.CommentReply = Backbone.Model.extend({
  url: function() {
    // /articles/:article_id/comments/:id/comment_replies
    return this.urlRoot;
  },

  initialize: function(options) {
    // Here we set this model url,
    // which is given by the Comment model through options,
    // which will have the dynamic path we need to reply
    // to the right comment in a given article.
    // /articles/:article_id/comments/1/comment_replies
    this.urlRoot = options.commentPath + '/comment_replies';
  }
});
{% endhighlight %}


I hope this gives you the answer you are looking for and I wish you happy coding.
