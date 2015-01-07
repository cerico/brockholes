---
layout: post
title: "ng animate"
description: ""
category: 
tags: [angularjs, ng-animate]
summary: In the last post a trailowner was able to administrate their trail, with the div containing the admin buttons becoming visible once $scope.adminstrating became true. Here we'll animate that transition. And we'll also animate between the different profile views
---
Here is our periscope div

1) app/assets/javascripts/templates/trail.html

{% highlight html%}

  <div class="periscope">
    <h1 class="trailname">{{trail.name}}</h1>
    <div class="periedit" ng-show="administrating">
       <form class="trailedit" ng-submit="editname()">
      <input type="text" placeholder="Trail Name" ng-model="newtrailname"  size="30">
      <input class="edit" type="submit" value="edit name">
      </form>
        <div class="editphotos"><a href="" ng-click="scrollTo('editphotos')">edit photos</a></div>
        <div class="editlocation"><a href="" ng-click="scrollTo('editmap')">edit location</a></div>
        <div class="editgpx" ng-click="deletegpx()" ng-show="ifgpx">delete gpx</div>
    </div>
  </div>
  {% endhighlight%}
  
2) regular view
  <img src="http://salterhebble.com/blogpics/view1.jpg">
  
3) adminstrating view
  <img src="http://salterhebble.com/blogpics/view2.jpg">
  
Normally only the trail name is visible, but when a trailowner is administrating a trail, the periedit div is shown, 

First lets get ng-animate

4) Bowerfile

{% highlight html%}
asset 'angular'
asset 'angular-ui-router' 
asset 'dropzone', "3.8.7"
asset 'angular-google-maps'
asset 'fancybox'
asset 'angular-animate'
{% endhighlight%}

5) terminal 

{% highlight html%}
➜  pennine git:(ng-animate) ✗ bundle exec rake bower:install
(in /Users/garethrobertlee/sites/scenicapp/pennine)
bower.js files generated
/usr/local/bin/bower install -p 
bower angular-animate#*         cached git://github.com/angular/bower-angular-animate.git#1.3.8
bower angular-animate#*       validate 1.3.8 against git://github.com/angular/bower-angular-animate.git#*
bower angular#1.3.8             cached git://github.com/angular/bower-angular.git#1.3.8
bower angular#1.3.8           validate 1.3.8 against git://github.com/angular/bower-angular.git#1.3.8
bower angular#>= 1.0.8          cached git://github.com/angular/bower-angular.git#1.3.8
bower angular#>= 1.0.8        validate 1.3.8 against git://github.com/angular/bower-angular.git#>= 1.0.8
bower angular#>=1.2.0           cached git://github.com/angular/bower-angular.git#1.3.8
bower angular#>=1.2.0         validate 1.3.8 against git://github.com/angular/bower-angular.git#>=1.2.0
bower angular#>= 1.0.2          cached git://github.com/angular/bower-angular.git#1.3.8
bower angular#>= 1.0.2        validate 1.3.8 against git://github.com/angular/bower-angular.git#>= 1.0.2
bower angular#1.x               cached git://github.com/angular/bower-angular.git#1.3.8
bower angular#1.x             validate 1.3.8 against git://github.com/angular/bower-angular.git#1.x
bower angular#1.3.8            install angular#1.3.8
bower angular-animate#*        install angular-animate#1.3.8

angular#1.3.8 bower_components/angular

angular-animate#1.3.8 bower_components/angular-animate
└── angular#1.3.8

➜  pennine git:(ng-animate) ✗ tree vendor/assets/bower_components/angular-animate 
vendor/assets/bower_components/angular-animate
├── README.md
├── angular-animate.js
├── angular-animate.min.js
├── angular-animate.min.js.map
├── bower.json
└── package.json

0 directories, 6 files
{%endhighlight%}

6) app/assets/javascripts/application.js

{%highlight javascript%}
//

//= require angular/angular
//= require angular-ui-router/release/angular-ui-router
//= require angular-rails-templates
//= require lodash/dist/lodash
//= require angular-google-maps/dist/angular-google-maps
//= require markerclusterer/markerclusterer
//= require jquery
//= require jquery_ujs
//= require angular-animate/angular-animate
//= require fancybox/source/jquery.fancybox
//= require_tree .
{%end highlight%}

7) app/assets/javascripts/app.js

{% highlight javascript%}
scenic = angular.module('scenic',['controllers','templates','ui.router','uiGmapgoogle-maps','ngAnimate'])
{%endhighlight%}

8) error
{%highlight javascript%}
angular.js:4103 Uncaught Error: [$injector:unpr] Unknown provider: $$jqLiteProvider <- $$jqLite <- $animate <- $compile
http://errors.angularjs.org/1.3.5/$injector/unpr?p0=%24%24jqLiteProvider%20%3C-%20%24%24jqLite%20%3C-%20%24animate%20%3C-%20%24compile
{%  endhighlight%}

this turned out to because my angular version and my angular-animate version weren't the same, so just updated the bowerfile and specify the version

9) Bowerfile

{% highlight html%}
asset 'angular'
asset 'angular-ui-router' 
asset 'dropzone', "3.8.7"
asset 'angular-google-maps'
asset 'fancybox'
asset 'angular-animate', "1.3.5"
{% endhighlight%}

10) terminal

{% highlight html%}
➜  pennine git:(animate135) ✗ bundle exec rake bower:install
bower.js files generated
/usr/local/bin/bower install -p 
bower angular-animate#1.3.5     cached git://github.com/angular/bower-angular-animate.git#1.3.5
bower angular-animate#1.3.5   validate 1.3.5 against git://github.com/angular/bower-angular-animate.git#1.3.5
bower angular-animate#1.3.5    install angular-animate#1.3.5

angular-animate#1.3.5 bower_components/angular-animate
└── angular#1.3.5
{% endhighlight%}

i have to say there was some caching stuff going on here i didnt really get, and it was still trying to use 1.3.8 instead of 1.3.5, i renamed the folder and it works, changed it back again and it tried to use 1.3.8 again, something to come back to here




