JS framworks for Scalable Application Architecture
============================================

Recently I watched the [Scalable JavaScript Application Architecture presentation by Nicholas Zakas](http://www.youtube.com/watch?v=mKouqShWI4o).
Other related resources:

* [Presentation summary](http://www.ubelly.com/2011/11/scalablejs/)
* [Presentation slides](http://www.slideshare.net/nzakas/scalable-javascript-application-architecture)
* [Patterns For Large-Scale JavaScript Application Architecture by Addy Osmani](http://addyosmani.com/largescalejavascript/)

The presentation is very interesting but it also leaves many open questions.
In short, the architecture contains following application layers:
* base library (jquery, etc)
* application core:
  * manages modules (register, tells when to start and when to stop)
  * handle errors (like wrap all modules' methods into try/catch and log errors)
  * enable inter-module communication
  * should be extensible (error handling, ajax wrapper, general utilites, anything!)
  * can use base library
* sandbox:
  * facade for modules above the core
  * interaction between modules via messages (events)
* modules (do not know about each other, only about sandbox)
  * call only own methods or sandbox methods
  * DOM access only inside own box (but do not use base library)
  * no access to non-native global objects, don't create global objects
  * ask sandbox for anything you need, don't reference other modules

Here are some questions raised by the presentation:
* It is stated that the architecture covers 'controller' part of MVC, but it is not clear how to plug M and V here. Should the module act as controller handle module / view interaction? If it is then it can be complex to do without base library. For me the 'module' here is more close to a widget content (some self-contained control with own box in the HTML and related js code) then to a controller from the MVC triad.
* Modules do not interact directly and only send messages to each other but this still does not mean absolute independence. Messages are sent with some data. And the receiver should expect some data structure. So in the case when this data changes it is necessary to review and fix all possible receivers.
* App core responsibilities are too wide (rememver "anything!") and the same is related to sandbox (because it acts as a facade for modules on top of the core).

Of cause these questions do not mean that architecture is not good, but it is definitely not full (and Nicholas mentions this).
It was interesting to see if there are frameworks build upon this idea.
And here is a list of frameworks I found (descriptions are taken from their sites):

  * [Aura](https://github.com/aurajs/aura) is a decoupled, event-driven architecture for developing widget-based applications.
  - [Hydra.js](http://tcorral.github.com/Hydra.js/) is the library that will help you to scale your app.
  - [Kernel.js](http://alanlindsay.me/kerneljs/) is an ultra lightweight (~4k) architecture for building scalable javascript applications.
  - [TerrificJS](http://terrifically.org/) provides you a Scalable Javascript Architecture, that helps you to modularize your jQuery/Zepto Code in a very intuitive and natural way.
  - [scaleApp](http://scaleapp.org/) is a tiny JavaScript framework for scalable One-Page-Applications / Single-Page-Applications.
  - [framework by Legalbox](https://github.com/legalbox/lb_js_scalableApp) - the Scalable JavaScript Application framework.
  - [AngularJS](http://angularjs.org/) - Superheroic JavaScript MVW Framework.
  - [Stackoverflow question on the topic](http://stackoverflow.com/questions/8701336/good-implementation-of-scalable-javascript-application-architecture-sandbox-by)

What was unexpected (even for me) is a presence of the AngualrJS here (more on this below).
Others are directly built on the ideas from Nicholas' presentation and most popular (looking at the number of stars on the github) is [Aura](https://github.com/aurajs/aura). Here are todo [application](https://github.com/sbellity/aura-todos) [examples](https://github.com/alexanderbeletsky/todomvc-aura) built on Aura.

So why AngularJS is here? Actually I do not have much experience with it, but from what I know it looks very close to the Nicholas' Scalable Architecture:
* base library - jQuery or angular's own jqLite implementation
* app core - angular itself
* sandbox - scope passed to the controller
 * along with the sandbox we can pass additional services to the controller (like $http for ajax requests), so the sandbox does not turn into God-object with too much responsibilities
* module - angular's controller
 * controllers are self-contained and sandboxed, can be build in pure JS without access to base library, interaction between modules - via events or via service which can act as a mediator

This way main parts of the scalable architecture are present here as well as other parts which are necessary to build an application (views, models, additional services).
