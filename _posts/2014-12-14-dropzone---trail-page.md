---
layout: post
title: "Dropzone   Trail Page"
description: ""
category: 
tags: [jquery,dropzone]
summary: "Using a dropzone to upload images to an existing trail"
---
Lets go back to our trail page and allow users to add photos to an existing trail, and for this I'm using the dropzone plugin, lets add it to the Bowerfile, install, and then require it in the application.js

<img src="http://salterhebble.com/blogpics/dropzone1.jpg">

<img src="http://salterhebble.com/blogpics/dropzone2.jpg">

<img src="http://salterhebble.com/blogpics/dropzone3.jpg">

(it wasnt immediately obvious where the dropzone had actually installed, just had to do a tree/ls to find it)

<img src="http://salterhebble.com/blogpics/dropzone4.jpg">


Firstly lets create a form/dropzone area, and wrap it in an ng-if so it shows when ready, we can grab the css from the dropzone site, and add it in to the app/assets/stylesheets folder - keeping it separate from our main css file


<img src="http://salterhebble.com/blogpics/dropzone5.jpg">

I have also made this background, in gimp, for the dropzone

<img src="http://salterhebble.com/s3/css/photouploadwide.png">

We'll need to post our photo(s) to eg /trails/5/photos, as these photos will belong to the trail we're on, so in order to do that we'll need to set up nested routes, which we can do as follows in the config/routes/rb


<img src="http://salterhebble.com/blogpics/photosroute.jpg">

Similarly, when we save the photo on the rails server it should have the trail data associated with it, ie it must belong_to a trail, so lets comment the default photo create (line 44) and replace with line 45 (we will also need to comment line 50 and add in line 51, or the server will return a 406 error, due to to the authenticity token being passed)

<img src="http://salterhebble.com/blogpics/photocontroller2.jpg">

Now lets return to the front end, and the trail.js.erb, to add in our dropzone configuration, to upload the photos

Normally our form would have a post action, but here it doesnt as we'll be defining that in the trail.js.erb, and at the same time we'll turn off dropzones autodiscover, as we'll define that ourselves. If we're on trail 5, this should post to /trails/5/photos, as weve already nested the photos route in config/routes.rb

<img src="http://salterhebble.com/blogpics/dropzone8.jpg">

and we'll define our dropzone as a function here, photoUpload()

<img src="http://salterhebble.com/blogpics/dropzone9.jpg">

This will define a new dropzone on the element with the id of media-dropzone, and will post to the URL defined in the $scope.posthere variable. We can then push the new photo into the $scope.trail.photos array to show immediately on the page, and on success we can remove the previews

we can also change any of the dropzone parameters here, so i've replaced the default message just with blank spaces, as we have our photoupload.png background which relays the info about the dropzone instead

<img src="http://salterhebble.com/blogpics/dropzone12.jpg">

Now we have dropzone uploads for an existing trail, we'll now need to do the same as part of a new trail, though this will be configured slightly differently, and we'll talk about that in the next post



