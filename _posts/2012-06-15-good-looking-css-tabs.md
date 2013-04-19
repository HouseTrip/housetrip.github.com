---
layout: post
title: Good looking shadowy CSS3 tabs
published: true
author: Dragos
author_role: Lead Developer
author_url: http://www.housetrip.com
author_avatar: http://www.gravatar.com/avatar/d054979fb9ca8599cfeb84936172dc28?s=36
summary: I'm going to show you guys how to create some nice looking tabs using CSS3 (sorry old IE guys!) Basically I didn't find a good solution online for good looking tabs that were easy to do and that also looked nice, so I ended up by creating our own Housetrip version.
---
I'm going to show you guys how to create some nice looking tabs using CSS3 (sorry old IE guys!)
Basically I didn't find a good solution online for good looking tabs that were easy to do and that also looked nice, so I ended up by creating our own Housetrip version.

This is the result we're searching for:
![basic layout](/images/2012-06-15/result.png)

##The basic set-up
We will start by creating the barebones of the **css** and of the **html**:
{% highlight css cssclass=medium_height_highlight %}
  #nav {
    margin: 0;
    padding: 0;
  }

  #nav li {
    list-style: none;
    display: inline-block;
    padding-right: 3px;
    border: 1px solid #000;
    }

  #nav li div {
    padding: 5px 15px;
    display: inline-block;
    font-size: 14px;
  }

  #tabbed_container {
    padding:10px;
    width:400px;
    height:200px;
    margin-top: 1px;
    border: 1px solid #000;
  }
{% endhighlight %}
This will create the following basic layout:
{% highlight html cssclass=medium_height_highlight %}
  <ul id="nav">
    <li><div>Tab1</div></li>
    <li><div>Tab2</div></li>
    <li><div>Tab3</div></li>
    <li><div>Tab4</div></li>
  </ul>
  <div id="tabbed_container">
    m dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat
  </div>
{% endhighlight %}
![basic layout](/images/2012-06-15/screenshot1.png)

I know it doesn't look too impresive, but we're getting there! Patience! Let's try to add more styles to make it more good looking...

##Going further
Let's start by adding some color to our tabs! And maybe remove that ugly border. I wish that the color is different for the opened tab or for the one you are hovering over.

Add the ***open*** **class** on the tab that you want it to be opened in the beginning:
{% highlight html cssclass=medium_height_highlight %}
  <li><div class="open">Tab1</div></li>
{% endhighlight %}

And now we can add some CSS for the colors:

{% highlight css cssclass=medium_height_highlight %}
  #nav li {
    list-style: none;
    display: inline-block;
    padding-right: 3px;
    }

  #nav li div {
    padding: 5px 15px;
    display: inline-block;
    font-size: 14px;
    background-color: #32B6C4;
    color: #fff;
  }

  #nav li div:hover, #nav li div.open {
    color: #000;
    background-color: #FFF;
  }
{% endhighlight %}
And look at that, now we have our little colored tabs that turn white when you hover over them:
![basic layout](/images/2012-06-15/screenshot2.png)

##Adding shadows
Now this is the CSS3 part, that will make the tabs really stick out from the rest of the crowd.
I'll start by adding a shadow to the container.

This is done by the **box-shadow** CSS attribute:
{% highlight css cssclass=medium_height_highlight%}
 #tabbed_container {
    padding:10px;
    width:400px;
    height:200px;
    margin-top: 1px;
    box-shadow: 0 0 5px #888;
  }
{% endhighlight %}
![basic layout](/images/2012-06-15/screenshot3.png)

I'll follow this by adding a shadow as well on the tab elements:
{% highlight css cssclass=medium_height_highlight%}
  #nav li div {
    padding: 5px 15px;
    display: inline-block;
    font-size: 14px;
    background-color: #32B6C4;
    color: #fff;
    box-shadow: 0 0 5px #888;
  }
{% endhighlight %}
![basic layout](/images/2012-06-15/screenshot4.png)
It looks much better than the version we started with but not enough.

Let's proceed further...

##Finishing touches
What we want is to **get rid of the ugly shadow** between the tab and the content somehow. I'll do this by doing a CSS **box shadow** trick.

Modify the css file for *hover* and *open* elements to look like this:
{% highlight css cssclass=medium_height_highlight%}
  #nav li div:hover, #nav li div.open {
    color: #000;
    background-color: #FFF;
    box-shadow: 0 5px 0 #fff, 0 0 5px #888;
  }
{% endhighlight %}
So to break it down, what **box-shadow: 0 5px 0 #fff, 0 0 5px #888;** does is actually adding two shadows. The **second** one goes around the element, and the **first** one, which is the *important* one, is a white shadow that spreads 5px down, *covering the other ugly shadows*.

In addition, I want that the non-opened tabs have a more discrete shadow, and that all the tabs have the top corners rounded, so lets modify the CSS accordingly:
{% highlight css cssclass=medium_height_highlight%}
  #nav li div {
    padding: 5px 15px;
    display: inline-block;
    font-size: 14px;
    background-color: #32B6C4;
    color: #fff;
    box-shadow:0 0 1px #333;
    border-top-left-radius: 5px ;
    border-top-right-radius: 5px ;
  }
{% endhighlight %}
For the rounded corners, i used the **border-top-left-radius** and **border-top-right-radius**.

Maybe you'll also want to get rid of the *height* attribute on the container, if that will suit you better. I will as well add a **pointer:cursor** to the tabs so that the cursor will change into the pointing hand.

In addition, here is the Javascript that will shuffle your tabs:
{% highlight javascript cssclass=medium_height_highlight%}
  function openedTabChanger() {
    $("#nav li div").live("click", function () {
      $("#nav li div").removeClass("open");
      $(this).addClass("open");
    })
  }
{% endhighlight %}
Load this function when the DOM finishes loading.

##That's it!
We have finished everything, so let's see the result again!
![basic layout](/images/2012-06-15/result.png)

##Sourcecode
You can find the sourcecode [here](http://jsfiddle.net/dmiron/JU5fx/1/)
