---
layout: post
title: "Angular Google Maps II"
description: ""
category: 
tags: [angular,google maps]
summary: "Using google maps to supply co-ordinates to a new trail"
---
#New Trail Page

First we can add the googlemap dependency to our new trail controller

<img src="http://salterhebble.com/blogpics/a1.jpg">

Then as before, make sure a map appears n the page

<img src="http://salterhebble.com/blogpics/a2.jpg">


<img src="http://salterhebble.com/blogpics/a3.jpg">

and as before, attach a marker, to make sure it appears on the page

<img src="http://salterhebble.com/blogpics/a5.jpg">

The big difference between this google map on the previous google maps we implemented, is that we want the marker to be set by the user as a click event, so we need to add a map event in the html

<img src="http://salterhebble.com/blogpics/a7.jpg">

and then in our controller, so first lets add the event, and just have it outputting to the console

<img src="http://salterhebble.com/blogpics/a6.jpg">

once thats working, lets have our click event create a marker, here the click bring back 3 attributes, and its the params we're interested in, as this provides the latitude and longitude, we can then push this marker into the empty array, and run $scope.$apply (as this is coming from outside of angular)

<img src="http://salterhebble.com/blogpics/a8.jpg">

In our case we only want one marker, as this will be the position of the trail, so here in line 23 we clear the array of previous markers on each click, replacing with the new position, until the user submits the trail

<img src="http://salterhebble.com/blogpics/a9.jpg">

and now we want to pass the latitude and longitude to angular, to form part of the form that will be submitted, so we can set that here in line 18 and 19

<img src="http://salterhebble.com/blogpics/a10.jpg">

and here it will form part of our form

<img src="http://salterhebble.com/blogpics/a11.jpg">





