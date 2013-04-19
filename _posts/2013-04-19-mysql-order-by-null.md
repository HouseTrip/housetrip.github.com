---
layout: post
title: Don't want MySQL filesort? Here's how to kill it dead!
published: true
author: Marcus
author_role: Lead Developer
author_url: http://github.com/marcusleemitchell
author_avatar: http://www.gravatar.com/avatar/c0afdc1c900a8f7b429f786507e19758?s=36
---
When using EXPLAIN to find the finer details of what is going on under the hood, you will surely come across this,

{% highlight mysql %}
  Using where; Using temporary; Using filesort
{% endhighlight %}

or some combination like that. The important things to notice here are `temporary` and `filesort`.  Fixing `temporary` requires carefully defining indices, more on that subject can be found [in the MySQL docs](http://dev.mysql.com/doc/refman/5.0/en/multiple-column-indexes.html).

Fixing the `filesort`, (especially when your SQL is abstracted by something like ActiveRecord) can be a little more tricky.  The problem comes from using ` ... :group => "[column]" ` in our scope, for example.

{% highlight ruby %}
  Booking.all(
    :select => "count(1) as bookings_per_day, date(created_at) as booking_date",
    :conditions => [
      "user_id = ?
      and status = 'accepted'
      and pending = 0
      and created_at between ? and ?",
      @user.id,
      extract_date(dates.first.beginning_of_week),
      extract_date(dates.last)
    ],
    :group => "booking_date")
{% endhighlight %}

The SQL query derived from this code results in a 'filesort' opertation, something we want to avoid. [MySQL docs](http://dev.mysql.com/doc/refman/5.0/en/order-by-optimization.html) state:

> By default, MySQL sorts all GROUP BY col1, col2, ... queries as if you specified ORDER BY col1, col2, ... in the query as well. If you include an ORDER BY clause explicitly that contains the same column list, MySQL optimizes it away without any speed penalty, although the sorting still occurs. If a query includes GROUP BY but you want to avoid the overhead of sorting the result, you can suppress sorting by specifying ORDER BY NULL.

**Bingo!**

{% highlight mysql %}
  INSERT INTO foo
  SELECT a, COUNT(*) FROM bar GROUP BY a ORDER BY NULL;
{% endhighlight %}

So let's go back to the code and add that in.

{% highlight ruby %}
  Booking.all(
    :select => "count(1) as bookings_per_day, date(created_at) as booking_date",
    :conditions => [
      .. etc ..
    ],
    :group => "booking_date",
    :order => "null")
{% endhighlight %}

*Note that it isn't `:order => nil`. This results in the order not being included in the result SQL at all.*

The EXPLAIN now gives us `Using where; Using temporary`. Chances are, the order implied by the `:group` is what you want anyway.  Of course, if you were sorting by a column explicitly you **would** care about the order, in which case you would be stating this in the query.  It's almost as if ActiveRecord should know to add this if there is no `:order` specified when using `:group`.

For more detail on exactly what MySQL is doing when filesorting, take a look at this detailed article, [How MySQL executes ORDER BY](http://s.petrunia.net/blog/?p=24)
