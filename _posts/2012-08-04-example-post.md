---
layout: post
title: Example Post Template (do not publish me)
published: false
author: Matt
author_role: Developer
author_url: https://plus.google.com/107981766773306423214?rel=author
author_avatar: http://www.gravatar.com/avatar/9e0e76de0fe26c1326da1a232d4dd2f2?s=36
summary:
---
Nulla facilisi. In vel sem. Morbi id urna in diam dignissim feugiat. Proin molestie tortor eu velit. Aliquam erat volutpat. Nullam ultrices, diam tempus vulputate egestas, eros pede varius leo, sed imperdiet lectus est ornare odio. Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Proin consectetuer velit in dui. Phasellus wisi purus, interdum vitae, rutrum accumsan, viverra in, velit. Sed enim risus, congue non, tristique in, commodo eu, metus. Aenean tortor mi, imperdiet id, gravida eu, posuere eu, felis. Mauris sollicitudin, turpis in hendrerit sodales, lectus ipsum pellentesque ligula, sit amet scelerisque urna nibh ut arcu. Aliquam in lacus. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Nulla placerat aliquam wisi. Mauris viverra odio. Quisque fermentum pulvinar odio. Proin posuere est vitae ligula. Etiam euismod. Cras a eros.

## Coming soon

![picture of a dog](images/dog.jpg)

Nulla facilisi. In vel sem. Morbi id urna in diam dignissim feugiat. Proin molestie tortor eu velit. Aliquam erat volutpat. Nullam ultrices, diam tempus vulputate egestas, eros pede varius leo, sed imperdiet lectus est ornare odio. Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Proin consectetuer velit in dui. Phasellus wisi purus, interdum vitae, rutrum accumsan. Cras a eros.

Nulla facilisi. In vel sem in hendrerit sodales, lectus ipsum pellentesque ligula, sit amet scelerisque urna nibh ut arcu. Aliquam in lacus. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Nulla placerat aliquam wisi. Mauris viverra odio. Quisque fermentum pulvinar odio. Proin posuere est vitae ligula. Etiam euismod. Cras a eros.

*   Posuere cubilia Curae; Nulla placerat aliquam wisi. Mauris viverra odio. Quisque fermentum pulvinar odio. Proin posuere est vitae ligula. Etiam euismod.  Nulla facilisi.
*   In vel sem in hendrerit sodales, lectus ipsum pellentesque ligula, sit amet scelerisque urna nibh ut arcu. Aliquam in lacus. Vestibulum ante ipsum primis in faucibus orci luctus et ultric
*   [Example link](http://example.com/ "With a Title")
*   Weibf wei wef

### Smaller title

Nulla facilisi. In vel sem in [example link](http://example.com/ "With a Title") sodales, lectus ipsum pellentesque ligula, sit amet scelerisque urna nibh ut arcu. Aliquam in lacus. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Nulla placerat aliquam wisi. Mauris viverra odio. Quisque fermentum pulvinar odio. Proin posuere est vitae ligula. Etiam euismod.  Nulla facilisi. In vel sem in hendrerit sodales, lectus ipsum pellentesque ligula, sit amet scelerisque urna nibh ut arcu. Aliquam in lacus. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Nulla placerat aliquam wisi. Mauris viverra odio. Quisque fermentum pulvinar odio. Proin posuere est vitae ligula. Etiam euismod. Cras a eros. Nulla facilisi. In vel sem in hendrerit sodales, lectus ipsum pellentesque ligula, sit amet scelerisque urna nibh ut arcu. Aliquam in lacus. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Nulla placerat aliquam wisi. Mauris viverra odio. Quisque fermentum pulvinar odio. Proin posuere est vitae ligula. Etiam euismo.

{% highlight ruby %}
  require 'rubygems'
  require 'ruby-prof'

  def slideview_image_url(photo)
    if CDN_OPTIONS && CDN_OPTIONS[:s3_host_alias]
      hosts = Array.wrap(CDN_OPTIONS[:s3_host_alias])
      host  = hosts[photo.id % hosts.size]
      "http://#{host}/photos/images/#{photo.id}/search2.jpg"
    else
      "/system/images/#{photo.id}/search2/#{photo.image_file_name}"
    end
  end

  def property_search_images_json_for_slideview(properties)
    properties.collect_hash do |property|
      photos     = property.photos.sort! { |a,b| (a.default_photo ==  b.default_photo) ? 1 : -1 }
      image_urls = photos.collect {|photo| slideview_image_url(photo) }
      [property.id, image_urls]
    end.to_json
  end
{% endhighlight %}

## Movie IFRAME

<iframe src="http://player.vimeo.com/video/2989396?title=0&amp;byline=0&amp;portrait=0&amp;color=fbff00" width="500" height="281" frameborder="0">texthastobehere</iframe>

#### Smallest title

Nulla facilisi. In vel sem in [example link](http://example.com/ "With a Title") sodales, lectus ipsum pellentesque ligula, sit amet scelerisque urna nibh ut arcu. Aliquam in lacus. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Nulla placerat aliquam wisi. Mauris viverra odio. Quisque fermentum pulvinar odio. Proin posuere est vitae ligula. Etiam euismod.  Nulla facilisi.
