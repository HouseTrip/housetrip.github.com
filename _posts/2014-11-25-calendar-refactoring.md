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

As you can see the calendar does not live *alone* but it's inserted in the context of a *search bar*. When you open the page the calendar is in its *inactive* state where only the calendar icon is visible, as shown here:

{% img /images/2014-11-25/calendar_closed_without_dates.png 300 %}

As soon as you click on the calendar icon the calendar goes into an *open state*. In such state the *date picker* is open and it's possible to select the *start date* of your journey.

{% img /images/2014-11-25/calendar_design.png 300 %}

When the user click on the start date (i) the start date box in the search bar is updated with the selected date, and (ii) the active date is moved to the end date.

{% img /images/2014-11-25/calendar_start_date_selected.png 300 %}

At this point the user can choose the end date, bringing the calendar to the following state:

{% img /images/2014-11-25/calendar_end_date_selected.png 300 %}

Notice that when the user is hovering a date but haven't chosen it yet, the corresponding box is highlighted with the hovered date like here:

{% img /images/2014-11-25/calendar_hovered_date.png %}

After the date selection is finished the *date picker* is closed and the search bar displays the selected dates in the corresponding boxes as follows:

{% img /images/2014-11-25/calendar_closed_with_dates.png 300 %}

Hopefully this is enough to give you an idea of the functionalities provided by this calendar. Let's now dig into the details of how this has been implemented, and why we decided to refactor it.

## The Big Monolith

As I sad the goal was to refactor the existing calendar, so let's have a look at the overall design of the existing component as I found it. To better understand it, let me enlarge the picture for one second and let's see how the overall search bar looks like:

{% img /images/2014-11-25/search_bar_design.png %}

there is one *backbone model* containing the informations regarding the search that must be performed, and three sub-views holding the responsibilities of the three visual elements in the bar. Up to here nothing surprising, but let's dig into the design of the calendar.

The public interface include methods to *activate()* and *deactivate()* the calendar. The calendar is *active* when it's the currently opened element in the *search bar* interface, *inactive* otherwise. *startDate()* and *endDate()* return the currently selected dates, while *clearDates()* reset your selection. *toggleCalendarManually()*, *toggleDatePickerManually()* and *toggleDatePickerVisibility()* to open/close the picker, and to set the calendar active/inactive states explicitely.

{% img /images/2014-11-25/monolith_public.png %}

The one above is arguably not the best public interface ever, but this is not our major problem. In fact the class itself is a monolith of **500+** lines of codes handling all the responsibilities related to the states that I described in the visual design. Here you can see the complete list of methods:

{% img /images/2014-11-25/monolith_full.png %}

This is what I usually define as an **iceberg class**. In the agile world this kind of classes arise quite easily. Developers start a component that is small and sounds reasonable, and each time a *story* adds a new requirements, a number of helpers methods are piled up hiding important responsibilities and cluttering the code. Empirically I would say that when the number of private methods becomes greater than 10, quite easily this is the principle of an *iceber class*. In this magnificent specimen of *iceberg class* we can count 44 private methods.

Back to scientific mode I usually rely on the following empirical classifications to determine how good a class is:

* **Highly Cohesive**: the concepts expressed in the class are tightly connected.
* **Loosely Coupled**: the class does not heavily relies on other ones.
* **Easily Composable**: the class facilitate composition.
* **Context Independent**: the class have no built-in knowledge about the system in which it executes.

These principles have been perfectly summarized in the <a href="http://vimeo.com/12350535">S. Metz talk</a> at <a href="http://goruco.com/">GORUCO</a> Conference. Coupling and Cohesion were actually invented by Larry Constantine in the late 1960s as part of <a href="http://books.google.co.uk/books/about/Structured_design.html?id=zMQmAAAAMAAJ">Structured Design</a> even before OOP was invented. Easily Composable and Context Independent have been added by Freeman S. and Pryce N. in <a href="http://books.google.co.uk/books/about/Growing_Object_Oriented_Software_Guided.html?id=QJA3dM8Uix0C">Growing Object-Oriented Software, Guided by Tests</a>. Simply said, <a href="http://onsmalltalk.com/objects-should-be-composable">objects should be like Legos</a>.

Arguably the calendar class that I just showed is violating most of these principle. It's definitely not highly cohesive since it contains terms from domains that do not strictly belong to the calendar like the private method *minimumStayIsSatisfied()*. Arguably it does not heavily violates the *Loosely Coupled* principle since the only explicit dependency it has is the <a href="http://foxrunsoftware.github.io/DatePicker/">Foxrun DatePicker</a> plugin it relies on. Definitely it is not context independent since it's public API assumes that the calendar lives inside the *search bar* and therefore has an active/inactive state. For the same reason the *Easily Composable* metric is heavily affected.

To complete the above assessment let's see what an empirical code quality librarys like <a href="https://github.com/es-analysis/plato">https://github.com/es-analysis/plato</a> tell us:

{% img /images/2014-11-25/calendar_old_complexity_gist1.png 600 %}

Of course this kind of analysis is not able to catch all the details that we got with the previous analysis based on software engineering principles. Nevertheless it is clear that there is an impact on *complexity* and *estimated probability of errors*. A more detailed report also highlights the most offensive methods:

{% img /images/2014-11-25/calendar_old_metrics.png 600 %}

## Refactoring

In order to get this calendar back in shape we need a plan. This is how I approached the problem:

* Identify and the **presentation layer**, the **application layer** and the **domain layer** (as explained in [DDD by E. Evans](http://goo.gl/V1nNa6))
* For each layer identify the responsibilities in the calendar and create objects for them

The *presentation layer* here is expressed in terms of Backbone views. Each backbone view is bound to a single element of the calendar. A deep visual analysis of the calendar actually reveals quite a number of entities that were "hidden" in the previous design and that are highlighted here:

{% img /images/2014-11-25/calendar_new_visual_components.png 600 %}

You can identify two main areas in the calendar: the *control* area and the *date picker* area. These will be represented in the code by simple package namespacing. Inside each area we can identify visual components with single responsibilities. For the *control* we have the *calendar icon*, the *dates boxes* and the *number of nights*. For the *date picker* we have the *dates* and the *remove dates* action. The *dates* are handled using the <a href="http://foxrunsoftware.github.io/DatePicker/">foxrun datepicker library</a>, so we don't have to go deeper than that. A trickier responsibility that I identified later was a backbone view that is responsible to make the picker visible or not, you will see the class in the design that follows shortly.

The *application layer* defines the jobs the software is supposed to do and directs the expressive domain objects to work out problems. To achieve this goals I created a *calendar controller* class that is responsible to (i) receive UI events, (ii) orchestrate the domain objects and (iii) notify the views of the updates. This is closely if not perfectly matching an MVC pattern, hence the name I gave to the class.

In the *domain layer* I identified two classed based on the high-level requirements for the calendar. The first one is obviously the dates. The second one is (less obviously) the *visibility* of the *control* and *picker* elements in the UI. At a first glance it may look like a view responsibility, but then working on the code I realised that one responsibility is *how* the visibility status is reflected in the DOM (i.e., the view) and another responsibility is the abstract status of the visual element itself (i.e., the domain model). This is definitely a not straightforward separation and I think it's open to interpretations.

Finally the original *calendar* class become a simple composition mechanism of the previous elements. The class diagram of what I just described is shown here:

{% img /images/2014-11-25/calendar_new_class_diagram.png 600 %}

The typical interaction starts from the user clicking on some UI element. The backbone view associated to this area of the DOM capture the event and triggers the corresponding UI event in the *HouseTrip.events* bus. The controller is subscribed to all the calendar events and work out what it should do for each of them orchestrating the domain objects. Finally, the controller trigger a *calendar_update* event to which the views are subscribed. This event provides a status read-only object that *serialise* the domain entities to be read by the view. This is a maybe unnecessary indirection that I introduced but I found it useful to separate the domain objects (unstable) from the information needed by the views to be rendered (quite stable in my experience). Here it is the status object in all its glory:

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

Please notice the business rule managed by the *controller*. If the user pick up a date and both the *from date* and *to date* fields are present, hide the picker. Arguably, the if statement in the controller is the responsibility of a *service object*, but in this case it's so simple that would be ridicolous to add it.

## Evaluate the new design

To evaluate the design let's go back to the principles. *Theoretically* I should pick every class and check that it is *Highly Cohesive*, *Loosely Coupled*, *Easily Composable* and *Context Independent*. I'll save you that and I just want to look at a couple of them. The first one is the calendar:

<pre class="lang:js decode:true " title="the new calendar class" >HouseTrip.Views.SearchBarCalendar = Backbone.View.extend({

  initialize: function(options) {
    this.behavior = new HouseTrip.SearchBarCalendarBehaviorController({
      model: this.model
    });

    this.calendarIcon = options.iconView || new HouseTrip.Views.SearchBarCalendarControlIcon({
      el: this.$el
    });

    this.dates = options.datesView || new HouseTrip.Views.SearchBarCalendarControlDates({
      el: this.$el
    });

    this.numOfNights = options.numOfNightsView || new HouseTrip.Views.SearchBarCalendarControlNumOfNights({
      el: this.$el
    });

    this.visibility = options.pickerVisibilityView || new HouseTrip.Views.SearchBarCalendarPickerVisibility({
      el: this.$el,
      calendarPosition: options.calendarPosition
    });

    this.dates = options.pickerDatesView || new HouseTrip.Views.SearchBarCalendarPickerDates({
      el: this.$el
    });

    this.removeDates = options.removeDateView || new HouseTrip.Views.SearchBarCalendarPickerRemoveDates({
      el: this.$el
    });

    this.legacyInteface = new HouseTrip.Views.SearchBarCalendarLegacyInterface({
      newCalendarView: this
    });
    this.delegateMethods(['activate', 'toggleCalendarManually', 'deactivate'], this.legacyInteface);

    new HouseTrip.Views.DatepickerFactory({ $el: this.$el }).make();

    this._afterDatepickerInitialization();
  },

  render: function() {
    this.behavior.triggerUpdate();
  },

  _afterDatepickerInitialization: function() {
    this.removeDates.render();
    this.dates.registerDatepickerCallbacks();
  }

});

_.extend(HouseTrip.Views.SearchBarCalendar.prototype, HouseTrip.Helpers.Delegation);
</pre> 

In this class there are some gotchas that I didn't discussed in the blog post because are irrelevant for what we are doing, specifically the *LegacyInterface* class and the *DatePickerFactory* class. The first one is needed to support some of the functionalities of the previous implementation while the second one is used to instantiate the foxrun datepicker.

Here, arguably, we achieved a pretty good degree of *cohesiveness* and *composability* which are order of magnitudes above the previous implementation. In terms of coupling this class is only creating the sub-views but actually does not rely on them, so I am also confident that this is *loosely coupled*. On the other hand I don't think this class is context independent and this is why is namespaced under *SearchBar*. 

Let's look at the controller now:

<pre class="lang:js decode:true " title="the calendar controller" >
HouseTrip.SearchBarCalendarBehaviorController = function(options) {
  this.initialize(options);
};

HouseTrip.SearchBarCalendarBehaviorController.prototype = _.extend({

  initialize: function(options) {
    this.model = options.model;

    this.dates = new HouseTrip.SearchBarCalendarBehaviorDates({
      fromDate: HouseTrip.Helpers.Dates.fromString(this.model.get('from_date')),
      toDate: HouseTrip.Helpers.Dates.fromString(this.model.get('to_date'))
    });

    this.visibility = new HouseTrip.SearchBarCalendarBehaviorVisibility({
      dates: this.dates
    });

    HouseTrip.Events.on('search_bar:calendar:activate', this._onActivate, this);
    HouseTrip.Events.on('search_bar:calendar:deactivate', this._onDeactivate, this);
    HouseTrip.Events.on('search_bar:calendar:show_control', this._onShowControl, this);

    HouseTrip.Events.on('search_bar:calendar:control:click_calendar_icon', this._onClickCalendarIcon, this);
    HouseTrip.Events.on('search_bar:calendar:control:click_from_date', this._onClickFromDate, this);
    HouseTrip.Events.on('search_bar:calendar:control:click_to_date', this._onClickToDate, this);

    HouseTrip.Events.on('search_bar:calendar:picker:pick_date', this._onPickDate, this);
    HouseTrip.Events.on('search_bar:calendar:picker:hover_date', this._onHoverDate, this);
    HouseTrip.Events.on('search_bar:calendar:picker:hide', this._onPickerHide, this);
    HouseTrip.Events.on('search_bar:calendar:picker:click_notice', this._onClickNotice, this);

    HouseTrip.Events.on('search_bar:calendar:page_scroll', this._onPageScroll, this);
  },

  triggerUpdate: function(event) {
    this.model.set('from_date', HouseTrip.Helpers.Dates.toSubmit(this.dates.fromDate()));
    this.model.set('to_date', HouseTrip.Helpers.Dates.toSubmit(this.dates.toDate()));

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

  _onClickFromDate: function() {
    this.visibility.show('picker');
    this.dates.selectFromDate();

    this.triggerUpdate();
  },

  _onClickToDate: function() {
    this.visibility.show('picker');
    this.dates.selectToDate();

    this.triggerUpdate();
  },

  _onHoverDate: function(date) {
    this.hoveredDate = date;

    this.triggerUpdate('onHoverDate');
  },

  _onPickerHide: function() {
    this.visibility.hide('picker');

    this.triggerUpdate();
  },

  _onPickDate: function(date) {
    this.dates.pickDate(date);

    if(this.dates.hasDates()) {
      this.visibility.hide('picker');
    }

    this.triggerUpdate('onPickDate');
  },

  _onActivate: function() {
    this.visibility.toggle(true);

    this.triggerUpdate('onActivate');
  },

  _onShowControl: function() {
    this.visibility.show('control');

    this.triggerUpdate('onActivate');
  },

  _onDeactivate: function() {
    this.visibility.toggle(false);

    this.triggerUpdate('onDeactivate');
  },

  _onClickCalendarIcon: function() {
    this.visibility.toggle();

    this.triggerUpdate('onClickCalendarIcon');
  },

  _onClickNotice: function() {
    this.visibility.toggle(false);
    this.dates.resetDates();

    this.triggerUpdate('onClickNotice');
  },

  _onPageScroll: function() {
    // nothing to do, just send back the status

    this.triggerUpdate('onPageScroll');
  },

}, Backbone.Events);
</pre> 

Also here there are some gotchas that I didn't covered while talking about the design. The biggest one is the input *model* that I receive as a paramenter in the initialize method and that I keep in sync when I trigger the update event. This is a dependency against the rest of the code in our front-end to actually submit the search. 

If we evaluate this code based on the previous principle arguably we can say that: (i) is highly cohesive, because all the verbs involved maps UI event to domain elements. Nothing more than that; (ii) is *somehow* loosely coupled since it heavily depends on the domain elements (expected), but it's not strictly tight with the views; (iii) *composability* and *context independence* are tricky to evaluate for controllers but in my opinion we didn't made any clear offense here. If you have comments on that I would be happy to hear them.

Finally, this is the view for the number of nights:

 
<pre class="lang:js decode:true " title="The number of nights view" >HouseTrip.Views.SearchBarCalendarControlNumOfNights = Backbone.View.extend({

  initialize: function(options) {
    this.translations = this.$el.data('translations');
    this.alwaysVisible = options.alwaysVisible !== undefined ? options.alwaysVisible : false;

    HouseTrip.Events.on('search_bar:calendar:update', this.render, this);
  },

  render: function(status) {
    if(!status) {
      return;
    }

    this._setNumberOfNights(status.fromDate, status.toDate);

    if(status.controlVisible || this.alwaysVisible) {
      this._open();

    } else {
      this._close();
    }
  },

  _setNumberOfNights: function(fromDate, toDate) {
    var difference = null;
    if(fromDate === null || toDate === null) {
      difference = 0;
    } else {
      difference = HouseTrip.Helpers.Dates.numberOfNights(toDate, fromDate);
    }
    if(difference &lt; 0) difference = 0;

    var nights = difference == 1 ? this.translations.inputs.night : this.translations.inputs.nights;
    var nightsWithNumber = _.template(nights)({ nights: difference });

    this.$('.number-of-nights').text(nightsWithNumber);
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

## Conclusions

TODO