Angular.js and SEO - pre-render content on the server
============================================

With angular.js you have an HTML which looks like this:

    <span>{{variableValue}}</span>
    <ul>
        <li ng-repeat="item in items" ng-bind="item.name"></li>
    </ul>

The simple way to make this content SEO-friendly is to pre-render data on the server and
then allow angular to do it's job on the client.
For simple variables there is [ng-bind](http://docs.angularjs.org/api/ng.directive:ngBind).
And for lists there is [ng-include](http://docs.angularjs.org/api/ng.directive:ngInclude).
Here is the example from above with pre-rendered content:

    <span ng-bind="variableValue">Static indexed value</span>
    <ul ng-include="'your/dynamic/list'">
        <li>seo-friendly item1</li>
        <li>seo-friendly item2</li>
    </ul>
    <script type="text/ng-template" id="your/dynamic/list">
        <li ng-repeat="item in items" ng-bind="item.name"></li>
    </script>

Other approaches to this problem include running angular on the server or crawling pages with a headless browser and
serve static pages to search bots (see links below).

Links
--------------------------------------------
[Pre-rendering AngularJS on the server with Node.js and jsdom](https://github.com/ithkuil/angular-on-server) and [discussion on reddit](http://www.reddit.com/r/javascript/comments/1a3fdj/prerendering_angularjs_on_the_server_with_nodejs/)

[angular and SEO discussion on google groups] (https://groups.google.com/forum/m/?fromgroups#!topic/angular/2uk2GYff28E)

[Ajax crawling - google developers] (https://developers.google.com/webmasters/ajax-crawling/docs/getting-started)

[AngularJS and SEO - approach based on PantomJS] (http://www.yearofmoo.com/2012/11/angularjs-and-seo.html) and [github repository](https://github.com/steeve/angular-seo)
