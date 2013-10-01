---
layout: post
title: "Interview: Geo Polygon Tools"
published: true
author: Dawid Sklodowski
author_role: Developer
author_url: https://github.com/dawid-sklodowski
author_avatar: http://www.gravatar.com/avatar/073c19a8d1fd30baa6dba34eaa55fe90.png
summary:  |

  Short interview with Developer and Online Performance
  Marketing Specialist about cool feature that our team recently has delivered.

---

  Interviewer: **Salim Halid**, interviewees: **Dawid Sklodowski**, **Carl Hallard**

___Salim Halid: Hi Dawid, introduce yourself to our readers.___

**Dawid Sklodowski**: Hey, I'm Dawid Sklodowski. I work at HouseTrip as Developer.

__SH: Recently your team developed a tool that allows us at HouseTrip to enhance our
geographical database with new areas. It’s a very interesting concept, so what made
you choose this project?__

**DS**: It seemed like a really interesting and challenging project; one that allowed us to
play a little more with Google Maps, draw with Javascript, utilise geometry algorithms
as well as the scaling of backend asynchronous processing.

__SH: What was the initial problem that needed to be solved?__

**DS**: Our different studies have highlighted how rural destinations are massively attractive
to our travellers - there are plenty of people within our target remit who are interested
in going on rural holidays or coastline holidays, not just big cities. HouseTrip offers
them the chance to do this, as such remote places don't usually have many hotels.

Geographical locations in our database are provided by a third parties and they only
contained administrative locations. We were lacking a healthy supply of informal regions,
which are often not even mentioned on maps or in official administrative structures.
To allow our guests to search for informal destinations, we needed to first input them
in to our database.

__HS: How did you solve the problem? Not even having the regions marked out on a map is a
pretty serious roadblock.__

**DS**: Exactly, so to be able to add those entities to our geo structure we needed a tool that
allowed us to draw an area on the map and then save it as a new destination.

__HS: And what tool did you use to implement drawing areas on the map?__

**DS**: We used the [Google Maps JavaScript API](http://developers.google.com/maps/documentation/javascript/)
which allowed us to draw and edit polygons on the map. When the polygon is saved,
our backend asynchronous processes examine the polygons and generate a list of every
HouseTrip properties contained within that area. The algorithm first looks for all of
the properties within the smallest rectangle which can contain drawn polygons and then,
for each of these, it is checking whether they are outside or inside the polygon.

<img src="/images/2013-10-01/lake-como.jpg" class="center-image" title="Lake Como" alt="Lake Como"/>
_An example of the polygon boundaries for [Lake Como](http://www.housetrip.com/en/lake-como), Italy_

After all of the properties are assigned to a new area, a landing page for that
destination is automatically generated. This allows our marketing team to drive
new traffic towards these new landing pages and efficiently smoke-test new locations.

__HS: In regards to property reassignment, how were you able to ensure that our current systems would be capable of handling such additional loads?__

**DS**: We drew a polygon that covered the whole of France as a measure of testing the impact
it would have on the performance of our delayed job workers. In a short while,
our servers managed to successfully load and process all 50,000+ properties we have
in France without affecting other jobs running on the servers.

__HS: Who are users of the tool you've built? Did you take into account their opinions
while developing it?__

**DS**: It’s being used by members of our Business Development team and our Marketing team.
During development we were cooperating with them tightly and showing early versions
to get their feedback on a regular basis.

__HS: Our guest today is also Carl Hallard, Online Performance Marketing Specialist
Host. Carl, how easy was it to use the tool?__

**Carl Hallard**: Very easy. It was very fast and efficient, enabling us to address demand for new
destinations in a few clicks. After drawing a polygon, a landing page for each
language was automatically created, and our guests could travel to their favourite
beaches, mountains and rural destinations in minutes.

__HS: How did you measure success? And on that note how successful was the product?__

**CH**: Measuring the success of this project was pretty straightforward; if the new
feature did its job it was a success, and if it didn’t, it wasn’t! The new polygon
tool enabled guests to access new destinations, and generated additional/incremental
bookings that we just wouldn't have generated otherwise.

Success!

__HS: Dawid, how successful was the project from technical point of view?__

**DS**: As we can see, HouseTrip users are happy, and so far we haven’t had a single
problem or performance decrease caused by our feature. Thus I believe it
was a success.

__HS: Thank you guys for this interview.__

**CH**: Thank you.

**DS**: Thank you.

<style type="text/css">
  .center-image {
    display:    block;
    margin:     1.4em auto !important;
    width:      670px;
    height:     auto;
  }
</style>