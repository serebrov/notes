Want Scalable Application Architecture? Check AngularJS.
============================================

The [Scalable JavaScript Application Architecture](http://www.youtube.com/watch?v=mKouqShWI4o) is a  presentation by [Nicholas Zakas](http://www.nczonline.net/) where hi suggests a flexible and scalable architecture. Here are other related resources:

* [Presentation summary](http://www.ubelly.com/2011/11/scalablejs/)
* [Presentation slides](http://www.slideshare.net/nzakas/scalable-javascript-application-architecture)
* [Patterns For Large-Scale JavaScript Application Architecture by Addy Osmani](http://addyosmani.com/largescalejavascript/)

The presentation is interesting but it also leaves many open questions.
In short, the architecture contains following application layers:
* base library (jquery, etc)
* application core:
  * manages modules (register modules, tell when to start and when to stop)
  * handle errors (like wrap all modules' methods into try/catch and log errors)
  * enable inter-module communication
  * should be extensible (error handling, ajax wrapper, general utilites, anything!)
  * can use base library
* sandbox:
  * facade for modules above the core
  * interaction between modules via messages (events)
* modules
  * do not know about each other, only about sandbox
  * call only own methods or sandbox methods
  * DOM access only inside own box (but do not use base library)
  * no access to non-native global objects, don't create global objects
  * ask sandbox for anything you need, don't reference other modules
  * preferably no access to base library, use pure JS

Here are some questions raised by the presentation:
* It is mentioned that the architecture covers 'controller' part of MVC, but it is not clear how to plug M and V here. Should the module act as controller and handle model / view interaction (this can be complex without access to the base library)? For me the 'module' here looks more like a [widget](http://en.wikipedia.org/wiki/Web_widget#Widget) - some self-contained element with own box in the HTML and related js code.
* Modules do not interact directly and only send messages to each other. This way we can change them independently. But messages are sent with some data and the receiver should expect some data structure. So in the case when this data changes it is necessary to review and fix all possible receivers.
* App core responsibilities are too wide (rememver "anything!") and the same is related to sandbox (because it acts as a facade on top of the core).

Of course good answers can be found for all these points and more questions can be raised. It is clear that suggested architecture is an idea and the way to go but not a complete design.
It was interesting to see if there are frameworks build upon this idea.
And here is the list (descriptions are taken from their sites):

  * [Aura](https://github.com/aurajs/aura) is a decoupled, event-driven architecture for developing widget-based applications.
  - [Hydra.js](http://tcorral.github.com/Hydra.js/) is the library that will help you to scale your app.
  - [Kernel.js](http://alanlindsay.me/kerneljs/) is an ultra lightweight (~4k) architecture for building scalable javascript applications.
  - [TerrificJS](http://terrifically.org/) provides you a Scalable Javascript Architecture, that helps you to modularize your jQuery/Zepto Code in a very intuitive and natural way.
  - [scaleApp](http://scaleapp.org/) is a tiny JavaScript framework for scalable One-Page-Applications / Single-Page-Applications.
  - [framework by Legalbox](https://github.com/legalbox/lb_js_scalableApp) - the Scalable JavaScript Application framework.
  - [AngularJS](http://angularjs.org/) - Superheroic JavaScript MVW Framework.
  - [Stackoverflow question on the topic](http://stackoverflow.com/questions/8701336/good-implementation-of-scalable-javascript-application-architecture-sandbox-by)

Presence of the AngualrJS here can be unexpected and reasons why I included it are below.
Others are directly built on the idea from Nicholas' presentation and most popular of them (looking at the number of stars on the github) is [Aura](https://github.com/aurajs/aura). Here are todo [application](https://github.com/sbellity/aura-todos) [examples](https://github.com/alexanderbeletsky/todomvc-aura) built on Aura.

So why AngularJS is here? Actually I do not have much experience with it, but from what I know it looks very close to the Nicholas' Scalable Architecture:
* base library - jQuery or angular's own jqLite implementation
* app core - angular itself
* sandbox - scope passed to the controller
* module - angular's controller

Besides the main parts of the scalable architecture AngularJS has more. For example, along with the sandbox (scope) we can pass additional services to the controller (like $http for ajax requests). So the sandbox does not turn into a God-object with too much responsibilities.

And angular controllers are very similar to scalable architecture modules:
* self-contained and sandboxed
* can be built in pure JS without access to base library
* interaction between modules is done via events or via special service but not directly

This way the idea of the scalable architecture is present in the AngularJS. And angular also gives other essential components (views, models, services, etc) and provides complete and ready to use architecture.
