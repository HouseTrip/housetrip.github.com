---
layout: post
title: Refactoring a complex front-end calendar
published: true
author: Alfredo Motta
author_role: Software Engineer
author_url: http://www.alfredo.motta.name
author_avatar: http://www.gravatar.com/avatar/c1044117a60a9c37a232cf8b6e2c87a8.png
summary: |
    Recently at HouseTrip I refactored the code of a complex frontend calendar. In this blog post I'll show you (i) how I translated the visual design into the code design, (ii) how I went beyond the basic Backbone.js patterns to make the calendar easier to understand and extend, and (iii) how the previous decisions affected the overall quality of the resulting code, hopefully using some kind of scientific metric.
---

## Introduction

Recently at HouseTrip I refactored the code of a complex frontend calendar. In this blog post I'll show you (i) how I translated the visual design into the code design, (ii) how I went beyond the basic Backbone.js patterns to make the calendar easier to understand and extend, and (iii) how the previous decisions affected the overall quality of the resulting code, hopefully using some kind of scientific metric.

## Visual design

HouseTrip is a revolutionary holiday rental website where you can book a whole home for less than the price of a hotel room. When you land on the homepage you get asked promptly to enter the **destination** of your holiday, the **dates** and the **number of people**. The date selection functionality is offered by the calendar I am presenting here.

{% img /images/2014-11-25/housetrip_home_page.png 300 %}

As you can see the calendar does not live *alone* but it's inserted in the context of a *search bar*. All the elements of the search bar can be *active* or *inactive*. An element is in its *active* state if the user is currently interacting with it, the element is in the *inactive* state otherwise. When you open the home page the calendar is in its *inactive* state. In this state only the calendar icon is visible, as shown here:

{% img /images/2014-11-25/calendar_closed_without_dates.png 300 %}

As soon as you click on the calendar icon the calendar goes into its *active* state. In such state the *date picker* is open and it's possible to select the *start date* of your journey as shown here:

{% img /images/2014-11-25/calendar_design.png 300 %}

When the user click on the start date the date box in the search bar is updated with the selected date and the active date is moved to the end date as shown here:

{% img /images/2014-11-25/calendar_start_date_selected.png 300 %}

At this point the user can choose the end date, bringing the calendar to the following state:

{% img /images/2014-11-25/calendar_end_date_selected.png 300 %}

Notice that when the user is hovering a date but haven't chosen it yet, the corresponding box is highlighted with the hovered date, as shown here:

{% img /images/2014-11-25/calendar_hovered_date.png %}

After the date selection is finished the *date picker* is closed and the search bar displays the selected dates in the corresponding boxes as follows:

{% img /images/2014-11-25/calendar_closed_with_dates.png 300 %}

Hopefully this is enough to give you an idea of the functionalities provided by this calendar. Let's now dig into the details of how this has been implemented, and why we decided to refactor it.

## The Iceberg Class

As I sad the goal was to refactor the existing calendar, so let's have a look at the overall design of the existing component as I found it. To better understand it, let me enlarge the picture for one second and let's see how the overall search bar looks like:

{% img /images/2014-11-25/search_bar_design.png %}

there is one *backbone model* containing the informations regarding the search that must be performed, and three sub-views holding the responsibilities of the three visual elements in the bar. Up to here nothing surprising, but let's dig into the design of the calendar.

The public interface include methods to *activate()* and *deactivate()* the calendar. The calendar is *active* when it's the currently opened element in the *search bar* interface, *inactive* otherwise. *startDate()* and *endDate()* return the currently selected dates, while *clearDates()* reset your selection. *toggleCalendarManually()*, *toggleDatePickerManually()* and *toggleDatePickerVisibility()* can be used to open/close the picker, and set the calendar to the active/inactive states explicitely.

{% img /images/2014-11-25/monolith_public.png %}

The one above is arguably not the best public interface ever, but this is not our major problem. In fact the class itself is a quite big entity of **500+** lines of codes handling all the responsibilities related to the states that I described in the visual design. Here you can see the complete list of methods:

{% img /images/2014-11-25/monolith_full.png %}

This is what I usually define as an **iceberg class**. It is my personal belief that in the agile world these kind of classes arise quite easily. Developers start a component that is small and sounds reasonable, and each time a *story* adds a new requirements, a number of helpers methods are piled up hiding important responsibilities and cluttering the code. Empirically I would say that when the number of private methods becomes greater than 10, quite easily this is the principle of an *iceber class*. In this magnificent specimen we can count 44 private methods.

## Code Quality Evaluation

To many people it is probably obvious why this class is not an acceptable implementation, but I would like to use well known metrics to actually prove it.
In order to assess the code quality I usually rely on the following empirical classification:

* **Highly Cohesive**: the concepts expressed in the class are tightly connected.
* **Loosely Coupled**: the class does not heavily relies on other ones.
* **Easily Composable**: the class facilitate composition.
* **Context Independent**: the class have no built-in knowledge about the system in which it executes.

These principles have been perfectly summarized in the <a href="http://vimeo.com/12350535">S. Metz talk</a> at <a href="http://goruco.com/">GORUCO</a> Conference. Coupling and Cohesion were actually invented by Larry Constantine in the late 1960s as part of <a href="http://books.google.co.uk/books/about/Structured_design.html?id=zMQmAAAAMAAJ">Structured Design</a> even before OOP was invented. Easily Composable and Context Independent have been added by Freeman S. and Pryce N. in <a href="http://books.google.co.uk/books/about/Growing_Object_Oriented_Software_Guided.html?id=QJA3dM8Uix0C">Growing Object-Oriented Software, Guided by Tests</a>. Simply said, <a href="http://onsmalltalk.com/objects-should-be-composable">objects should be like Legos</a>.

Arguably the calendar class that I just showed is violating most of these principle. It is definitely not highly cohesive since it contains terms from domains that do not strictly belong to the calendar like the private method *minimumStayIsSatisfied()*. Arguably it does not heavily violates the *Loosely Coupled* principle since the only explicit dependency it has is the <a href="http://foxrunsoftware.github.io/DatePicker/">Foxrun DatePicker</a> plugin it relies on. Definitely it is not context independent since it's public API assumes that the calendar lives inside the *search bar* and therefore has an active/inactive state. For the same reason the *Easily Composable* metric is heavily affected.

To complete the above assessment let's see what an empirical code quality librarys like <a href="https://github.com/es-analysis/plato">https://github.com/es-analysis/plato</a> tell us:

{% img /images/2014-11-25/calendar_old_complexity_gist1.png 600 %}

Plato is not able to catch the soft details that we made explicit with the previous analysis. Nevertheless it is clear that there is an impact on *complexity* and *estimated probability of errors*. A more detailed report also highlights that the most offensive methods are the *render()* method (SLOC = 35) and the *onDatepickerHoverDate()* method (SLOC = 9).

## Refactoring

In order to get this calendar back in shape we need a plan. This is how I approached the problem:

* Identify and the **presentation layer**, the **application layer** and the **domain layer**.
* For each layer identify a set of single responsibilities classes.
* Design the interactions between the classes trying to constantly minimize the dependencies between them.

The concepts of *presentation layer*, *application layer* and *domain layer* have been perfectly described in [Domain Driven Design by E. Evans](http://goo.gl/V1nNa6). It is definitely a general purpose technique and can be applied to your software no matter the language you use and (often) the granularity of the code you are working on. The *presentation layer* in the calendar is expressed in terms of Backbone views. Each backbone view is bound to a single visual element of the calendar. By carefully analyze the calendar we can actually find quite a number of entities that were "hidden" in the previous design and that are now highlighted here:

{% img /images/2014-11-25/calendar_new_visual_components.png 600 %}

You can identify two main areas in the calendar: the *control* area and the *date picker* area. These will be represented in the code by simple package namespacing. Inside each area we can identify visual components with single responsibilities: for the *control* we have the *calendar icon*, the *dates boxes* and the *number of nights*; for the *date picker* we have the *dates* and the *remove dates* action. The *dates* are handled using the <a href="http://foxrunsoftware.github.io/DatePicker/">foxrun datepicker library</a>, so we don't have to go deeper than that. A trickier responsibility that I identified later was a backbone view that is responsible to make the picker visible or not, you will see the class in the design that follows shortly.

The *application layer* defines the jobs the software is supposed to do and directs the expressive domain objects to work out problems. To achieve this goals I created a *calendar controller* class that is responsible to (i) receive UI events, (ii) orchestrate the domain objects and (iii) notify the views of the updates. This is closely if not perfectly matching an MVC pattern, hence the name I gave to the class. This is also the moment in which I needed to carefully think about the communication between the presentation layer and the application layer. For this boundary I wanted the highest degree of decoupling available since the change between this border is very likely to happen. Hence I went through the path of a pub/sub event dispatching mechanism.

In the *domain layer* I identified two classed based on the high-level requirements for the calendar. The first one is obviously the dates. The second one is (less obviously) the *visibility* of the *control* and *date picker* elements in the UI. At a first glance it may look like a view responsibility, but then working on the code I realised that one responsibility is *how* the visibility status is reflected in the DOM (i.e., the view) and another responsibility is the abstract status of the visual element itself (i.e., the domain model). This is definitely a not straightforward separation and I think it's open to interpretations.

Finally the original *calendar* class now have the responsibility to compose the previously presented elements. The complete class diagram of the design is shown here:

{% img /images/2014-11-25/calendar_new_class_diagram.png 600 %}

The typical interaction starts from the user clicking on some UI element. The backbone view associated to this area of the DOM capture the UI event and triggers the corresponding pub/sub event in the *HouseTrip.events* bus. The controller is subscribed to all the calendar events and work out what it should do for each of them orchestrating the domain objects. Finally, the controller trigger a *calendar:update* event to which the views are subscribed. This event provides a status read-only object that *serialise* the domain entities to be read by the view. This is a maybe unnecessary indirection that I introduced but I found it useful to separate the domain objects from the information needed by the views to be rendered. This is definitely an arguable design decision and the rationale behind it is to further decrese the coupling between the layers. Here it is the status object in all its glory:

<pre>  
  var status = {
    active: this.visibility.isVisible('active'),
    controlVisible: this.visibility.isVisible('control'),
    pickerVisible: this.visibility.isVisible('picker'),
    fromDate: this.dates.fromDate(),
    toDate: this.dates.toDate(),
    onFromDate: this.dates.is('onFromDate'),
    onToDate: this.dates.is('onToDate'),
    hoveredDate: this.hoveredDate,
    hovering: event == 'onHoverDate'
  };
</pre>

An example of interactions between these objects is shown in the sequence diagram underneath which starts when the user select a date from the picker panel:

{% img /images/2014-11-25/calendar_new_sequence_diagram.png 600 %}

Please notice the business rule managed by the *controller*: if the user pick up a date and both the *from date* and *to date* fields are present, hide the picker. Arguably, the if statement in the controller is the responsibility of a *service object*, but in this case it's so simple that would be ridicolous to add it.

## Evaluate the new design

To evaluate the design let's go back to the principles and evaluate the degree of cohesiveness, coupling, composability and context independence for some key classes we produced. The first one is the calendar which simply acts as a composition mechanism. Here in the constructor we can see the elements we discussed:

<pre class="lang:js decode:true " title="the new calendar class" >HouseTrip.Views.SearchBarCalendar = Backbone.View.extend({
  initialize: function() {
    this.calendarIcon = new HouseTrip.Views.SearchBarCalendarControlIcon({
      el: this.$('.calendar-icon')
    });

    this.dates = new HouseTrip.Views.SearchBarCalendarControlDates({
      el: this.$('.calendar-control-dates')
    });

    this.numOfNights = new HouseTrip.Views.SearchBarCalendarControlNumOfNights({
      el: this.$('.calendar-control-nights')
    });

    this.visibility = new HouseTrip.Views.SearchBarCalendarPickerVisibility({
      el: this.$('.date-picker')
    });

    this.dates = new HouseTrip.Views.SearchBarCalendarPickerDates({
      el: this.$('.date-picker')
    });

    this.removeDates = new HouseTrip.Views.SearchBarCalendarPickerRemoveDates({
      el: this.$('.date-picker')
    });

    [...]
  },

  render: function() {
    this.behavior.triggerUpdate();
  }

  [...]
});

_.extend(HouseTrip.Views.SearchBarCalendar.prototype, HouseTrip.Helpers.Delegation);
</pre> 

Here, arguably, we achieved a pretty good degree of *cohesiveness* and *composability* which are order of magnitudes above the previous implementation. In terms of coupling this class is only creating the sub-views but actually does not rely on them, so I am also confident that this is *loosely coupled*. On the other hand I don't think this class is context independent and this is why is namespaced under *SearchBar*. 

Let's look at the controller now:

<pre class="lang:js decode:true " title="the calendar controller" >
HouseTrip.SearchBarCalendarController = function() {
  this.initialize();
};

HouseTrip.SearchBarCalendarController.prototype = _.extend({

  initialize: function() {
    this.dates = new HouseTrip.SearchBarCalendarDates();
    this.visibility = new HouseTrip.SearchBarCalendarVisibility();

    HouseTrip.Events.on('search_bar:calendar:activate', this._onActivate, this);
    HouseTrip.Events.on('search_bar:calendar:deactivate', this._onDeactivate, this);

    HouseTrip.Events.on('search_bar:calendar:control:click_calendar_icon', this._onClickCalendarIcon, this);
    HouseTrip.Events.on('search_bar:calendar:control:click_from_date', this._onClickFromDate, this);
    HouseTrip.Events.on('search_bar:calendar:control:click_to_date', this._onClickToDate, this);

    HouseTrip.Events.on('search_bar:calendar:picker:pick_date', this._onPickDate, this);
    HouseTrip.Events.on('search_bar:calendar:picker:hover_date', this._onHoverDate, this);
    HouseTrip.Events.on('search_bar:calendar:picker:hide', this._onPickerHide, this);
    HouseTrip.Events.on('search_bar:calendar:picker:click_notice', this._onClickNotice, this);
  },

  triggerUpdate: function(event) {
    var status = {
      active: this.visibility.isVisible('active'),
      controlVisible: this.visibility.isVisible('control'),
      pickerVisible: this.visibility.isVisible('picker'),
      fromDate: this.dates.fromDate(),
      toDate: this.dates.toDate(),
      onFromDate: this.dates.is('onFromDate'),
      onToDate: this.dates.is('onToDate'),
      hoveredDate: this.hoveredDate,
      hovering: event == 'onHoverDate'
    };

    HouseTrip.Events.trigger('search_bar:calendar:update', status);
  },

  [...]

  _onPickDate: function(date) {
    this.dates.pickDate(date);

    if(this.dates.hasDates()) {
      this.visibility.hide('picker');
    }

    this.triggerUpdate('onPickDate');
  }

  [...]
}, Backbone.Events);
</pre> 

I omitted most of the functions since they all do more all less what the *_onPickDate()* method is doing. If we evaluate this code based on the previous principle arguably we can say that: (i) is highly cohesive, because all the verbs involved maps UI event to domain elements. Nothing more than that; (ii) is *somehow* loosely coupled since it heavily depends on the domain elements (expected), but it's not strictly tight with the views; (iii) *composability* and *context independence* are tricky to evaluate for controllers but in my opinion we didn't made any clear offense here. If you have comments on that I would be happy to hear them.

Finally, this is the view for the number of nights:

 
<pre class="lang:js decode:true " title="The number of nights view" >HouseTrip.Views.SearchBarCalendarControlNumOfNights = Backbone.View.extend({

  initialize: function() {
    HouseTrip.Events.on('search_bar:calendar:update', this.render, this);
  },

  render: function(status) {
    if(!status) {
      return;
    }

    this._setNumberOfNights(status.fromDate, status.toDate);

    if(status.controlVisible) {
      this._open();

    } else {
      this._close();
    }
  },

  _setNumberOfNights: function(fromDate, toDate) {
    [..compute the difference..]

    this.$('.number-of-nights').text(numberOfNightsText);
  },

  _close: function() {
    this.$('.number-of-nights').hide();
  },

  _open: function() {
    this.$('.number-of-nights').show();
  }

});
</pre> 

Here I truly believe you can see the advantage of the approach. The view is *highly cohesive* because it only speaks the language of the DOM. It is *loosely coupled* since it only relies on the *status* interface. It is *context independent* because no matter where you place this view it simply needs the event to be published, and finally is *loosely coupled* simply because it does not strictly rely on other objects.

To wrap up this evaluation let's see the updated stats that Plato provides us:

{% img /images/2014-11-25/plato_new_calendar.png %}

We can make some simple observations on these data: 

* The SLOC are more equally distributed between the different classes. The *controller* and the *control dates* being the exception with slightly bigger numbers. 
* The reported complexity is always low, apart from the *control dates* view.
* In general it *seems* (with some degree of mercy) that the classes follow the <a href="http://robots.thoughtbot.com/sandi-metz-rules-for-developers">Sandi Metz rules</a> regarding the complexity. No more than a 100 lines for each class, no more that 5 lines for each method.

You can argue that the number of classes is definitely increased and that a newcomer to the code probably needs some minutes to review it's organization before being able to change it. Despite that I truly believe this design is easier to work with, easier to maintain, and easier to test (which is an entire different blog post, but you may already guess the advantages on this area).

## Conclusions

TODO
