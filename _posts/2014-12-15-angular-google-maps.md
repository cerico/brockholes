---
layout: post
title: "Angular Google Maps"
description: ""
category: 
tags: [angular,google,maps]
---

#Trail Page

Lets replace our static map with a dynamic map, angular-google-maps. We can install this with bower as before

1)

<img src="http://salterhebble.com/blogpics/trailmap1.jpg">

2)

<img src="http://salterhebble.com/blogpics/trailmap2.jpg">

3)

<img src="http://salterhebble.com/blogpics/trailmap3.jpg">

4)

<img src="http://salterhebble.com/blogpics/trailmap4.jpg">

Then add it to the app

5)

<img src="http://salterhebble.com/blogpics/trailmap.jpg">

We'll be using this first on the trail page, so lets add it to the trailscontroller

6)

<img src="http://salterhebble.com/blogpics/trailmap5.jpg">

Now we'll need the Google Maps API, so we can go to the developer console, activate it... 

7)

<img src="http://salterhebble.com/blogpics/mapkey1.jpg">

8)

<img src="http://salterhebble.com/blogpics/mapkey3.jpg">

9)

<img src="http://salterhebble.com/blogpics/mapkey4.jpg">

10)

<img src="http://salterhebble.com/blogpics/mapkey5.jpg">

11)

<img src="http://salterhebble.com/blogpics/mapkey6.jpg">

and put it into the app

12)

<img src="http://salterhebble.com/blogpics/mapkey7.jpg">

lets replace the static map on line 58, with the angular map, on line 56

13)

<img src="http://salterhebble.com/blogpics/gm1.jpg">

not forgetting to add some css for the class it uses

14)

<img src="http://salterhebble.com/blogpics/gm2.jpg">

hardcode a map with a static location to make sure it comes out right

15)

<img src="http://salterhebble.com/blogpics/gm3.jpg">

and replace with the trails co-ordinates

16)

<img src="http://salterhebble.com/blogpics/gmap4.jpg">

#Map Page

We have a map page, which is designed to show all available trails on one map, so lets look at the configuration for that

17)

<img src="http://salterhebble.com/blogpics/mappage1.jpg">

I've handled this page slightly differently, as i had problems with images in the info-window if i structured the js as i had in the trail page, so this is an area for refactoring, as this feels rather unwieldy and cobbled together.

I got the markerclusterer from here, and downloaded it to /vendor/assets/bower_components/markerclusterer/markerclusterer.js, 

[http://google-maps-utility-library-v3.googlecode.com/svn/trunk/markerclusterer/src/markerclusterer.js](http://)

18)

<img src="http://salterhebble.com/blogpics/mc.jpg">

then i required it here, I did this to try keep a kind of consistency with the other front end assets, as though it really had come from Bower


19)

<img src="http://salterhebble.com/blogpics/mappage2.jpg">

so here ive created a new map, with center and zoom options, an empty markers array, and then we iterate through the trails, creating a new marker for each, and then pushing into the markers array

20) 

<img src="http://salterhebble.com/blogpics/mappage3.jpg">

Here we have an event listener appended to each marker, that will pop up an infowindow, showing details about that trail

21/22)

<img src="http://salterhebble.com/blogpics/mappage4.jpg">

<img src="http://salterhebble.com/blogpics/mappage5.jpg">

redrawing the map based on the search result

23)

<img src="http://salterhebble.com/blogpics/mappage6.jpg">

and when the page has been redrawn, we can add up all the markers on the new map, and then output that in the html

24)

<img src="http://salterhebble.com/blogpics/mappagelast.jpg">






