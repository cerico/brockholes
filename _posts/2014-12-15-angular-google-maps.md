---
layout: post
title: "Angular Google Maps"
description: ""
category: 
tags: [angular,google,maps]
---

#Trail Page

Lets replace our static map with a dynamic map, angular-google-maps. We can install this with bower as before


<img src="http://salterhebble.com/blogpics/trailmap1.jpg">

<img src="http://salterhebble.com/blogpics/trailmap2.jpg">

<img src="http://salterhebble.com/blogpics/trailmap3.jpg">

<img src="http://salterhebble.com/blogpics/trailmap4.jpg">

Then add it to the app

<img src="http://salterhebble.com/blogpics/trailmap.jpg">

We'll be using this first on the trail page, so lets add it to the trailscontroller

<img src="http://salterhebble.com/blogpics/trailmap5.jpg">

Now we'll need the Google Maps API, so we can go to the developer console, activate it... 

<img src="http://salterhebble.com/blogpics/mapkey1.jpg">

<img src="http://salterhebble.com/blogpics/mapkey3.jpg">

<img src="http://salterhebble.com/blogpics/mapkey4.jpg">

<img src="http://salterhebble.com/blogpics/mapkey5.jpg">

<img src="http://salterhebble.com/blogpics/mapkey6.jpg">

and put it into the app

<img src="http://salterhebble.com/blogpics/mapkey7.jpg">

lets replace the static map on line 58, with the angular map, on line 56

<img src="http://salterhebble.com/blogpics/gm1.jpg">

not forgetting to add some css for the class it uses

<img src="http://salterhebble.com/blogpics/gm2.jpg">

hardcode a map with a static location to make sure it comes out right

<img src="http://salterhebble.com/blogpics/gm3.jpg">

and replace with the trails co-ordinates

<img src="http://salterhebble.com/blogpics/gmap4.jpg">

#Map Page

We have a map page, which is designed to show all available trails on one map, so lets look at the configuration for that

<img src="http://salterhebble.com/blogpics/mappage1.jpg">

We'll drop our map in with the ui-gmap-googlemap tags, the important attributes here being zoom and center, and then we can add the markers inside the map tags, the important ones to note being the Markers attribute, and the clustermarkers/clusteroptions. We also have a click event to open up a ui-gmap-window when a marker is clicked

<img src="http://salterhebble.com/blogpics/mappage2.jpg">

In the angular controller, we can see the center and zoom options, and that map.Markers is set as an empty array. In line 28 we iterate through each trail and create a marker object, which is then pushed into the array. We also pass id,name and photo as part of the object, which can then be seen in line 24 of the html





