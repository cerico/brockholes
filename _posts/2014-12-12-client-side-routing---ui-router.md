---
layout: post
title: "Client Side Routing with ui-router"
description: ""
category: 
tags: [ui-router,angular]
---
We've introduced angular to our app, but the views are still being handled by Rails, we can reduce trips to the server by moving our routing to the front end, and injecting templates. I originally did this with the standard ng-route, but here i'm going to be using ui-router, so lets first add that to our Bowerfile and install

<img src="http://salterhebble.com/blogpics/ui1.jpg">

<img src="http://salterhebble.com/blogpics/ui2.jpg">

then add the angular-rails-templates gem 

<img src="http://salterhebble.com/blogpics/uigem.jpg">

then we can require ui-router and angular templates in the application.js


<img src="http://salterhebble.com/blogpics/ui3.jpg">

Now let replace our rails <% yield %> with ui-view, which is where our angular templates will get injected, here they are in layouts/application.html.erb

<img src="http://salterhebble.com/blogpics/ui4.jpg">

and in our app.js.erb, add in templates and the ui.router as dependencies

<img src="http://salterhebble.com/blogpics/ui5.jpg">

and we can start our router 

<img src="http://salterhebble.com/blogpics/ui6.jpg">

(above you can see that the controllers for each template are being defined here, in the main app.js.erb, which means we should remove the ng-controller from the templage page itself - so lets go ahead and delete that, otherwise we're likely to be doubling up any trips to the server)

<img src="http://salterhebble.com/blogpics/remove.jpg">

so we'll use $stateProvider, $urlRouterProvider and $locationProvider, and we'll define two routes for now, /trails, for our trails index page, and /trails/:id for the trails#show pages, and they'll come under the controllers we've previously written for them. We'll also make a default url, that'll take us to the trails index page. Now lets create those two templates, in app/assets/javascripts/templates, and call them trailsindex.html and trail.html, and we'll just output the following on both to make sure they hook to our router and controllers correctly

<img src="http://salterhebble.com/blogpics/ui7.jpg">

and if this outputs correctly, we can cut and paste the html from our trails#index page, and put it in our new trailsindex.html template, and then we can do the same with our trails#show html, putting it into the trail template, but one thing has to change here, we should now make sure our GET method in the getTrailInfo function uses location.path() instead of location.$$absUrl

<img src="http://salterhebble.com/blogpics/ui8.jpg">

We should now change our links, and add a hashbang for all angular routes

Before we go ahead and do the profile, new and map pages, lets make sure this is going to work on dokku and heroku. Lets replace config.assets.precompile=false in the config/application.rb with

<img src="http://salterhebble.com/blogpics/precompile.jpg">

and we'll also need to exclude the map and zip files from our repository, to avoid dokku issues 

<img src="http://salterhebble.com/blogpics/gitrm.jpg">


Now lets go back to our app, and create the templates for the rest of our pages

<img src="http://salterhebble.com/blogpics/ui12.jpg">

so here we've created routes for profile, new trail and map templates, so now lets go and create those in app/assets/javascripts/templates, and copy and paste our html from the user#show and trail#new views, and also the html from the dummy app for the map page

<img src="http://salterhebble.com/blogpics/ui13.jpg">

we already have a profile controller in profile.js.erb, so lets create a new.js.erb and a map.js.erb to go along with that. And change the links in the navbar to be angular links, with the hashbang in place

<img src="http://salterhebble.com/blogpics/links.jpg">

The profile page should work straightaway as we have already written the controller for it, but as before the http GET request shoud change to location.path()

<img src="http://salterhebble.com/blogpics/ui15.jpg">

lets also remove any unneccessary links from the profile template, and convert the photolinks to trails to angular links, like so

<img src="http://salterhebble.com/blogpics/lastone.jpg">


Now our new trail template still has a rails form there, shown below, which we'll need to change

<img src="http://salterhebble.com/blogpics/railsform.jpg">

to 

<img src="http://salterhebble.com/blogpics/angform.jpg">

We're going to include a dropzone here, so we'll leave the newtrail template here for now, and cover that in the next post







