---
layout: post
title: "GPX wth Nokogiri"
description: ""
category: 
tags: [rails,nokogiri,paperclip,gpx]
---

#Adding gpx to the new trail form

We need to be able to display route information on the trail map, via gpx. I did this using this method as a starting point, but it needed a number of modifications to make it work correctly

[http://larsgebhardt.de/parse-xml-with-ruby-on-rails-paperclip-and-nokogiri/](http://)

I decided to stay with paperclip rather than using carrierwave, to keep things separate as carrierwave is handling image uploads, though I will look later at using Carrierwave for both images and gpx. Some of the following is also detailed in the above link, though there are a number of difference to make it work with this apps set up



Firstly, add in the paperclip and nokogiri gems

1)

<img src="http://salterhebble.com/blogpics/gx1.jpg">



We'll need to add attachment to trail in the migration 

2)

<img src="http://salterhebble.com/blogpics/gx2.jpg">

3)

<img src="http://salterhebble.com/blogpics/gx3.jpg">

4)

<img src="http://salterhebble.com/blogpics/gx4.jpg">

then create the Tracksegment and Point models 

5)

<img src="http://salterhebble.com/blogpics/gx5.jpg">

6)

<img src="http://salterhebble.com/blogpics/gx6.jpg">

the point model shouldnt need modification

7)

<img src="http://salterhebble.com/blogpics/gx7.jpg">

the tracksegment we need to add a has_many relation

8)

<img src="http://salterhebble.com/blogpics/gx8.jpg">

and the trail model needs has_many for tracksegments, and a has_many through for points, an attached_file for gpx, and a parse_file method before save, which we'll then need to write

9)

<img src="http://salterhebble.com/blogpics/gx9.jpg">

the parse_file method taken from the link above and shown below only handled certain types of gpx file and didnt process the gpx file i was originally trying with, so im not sure how many different formats gpx files come in, below ive modified a couple of the methods (appended with a 2), in order for it to work with a second form of gpx, but this looks to be an area for further investigation and testing with different gpx files. Lets also wrap the parse_gpx method in an if statement as most trails will probably be uploaded without gpx data

10)

<img src="http://salterhebble.com/blogpics/gx10.jpg">

11)

<img src="http://salterhebble.com/blogpics/gx11.jpg">


Now we need to add a field in our form for the user to attach the gpx

12)

<img src="http://salterhebble.com/blogpics/gx12.jpg">

13)

<img src="http://salterhebble.com/blogpics/gx13.jpg">


which might need a bit of styling, but we can come back to that. Lets put a binding.pry in the trails controller create method, and upload a trail with some gpx and see what happens

14)

<img src="http://salterhebble.com/blogpics/gx14.jpg">

Normally the form would have a post action with a destination, but we're passing our trail through the dropzonejs. So it stuck me then, a gpx is a file just the same as an image file, and can be added to the dropzone just the same, we can also get rid of the extra upload button that way. so lets remove the input tag for the gpx, and try adding a trail with an image and a gpx file 

15)

<img src="http://salterhebble.com/blogpics/gxb1.jpg">

so here we can see the rails server receiving two files in the params but with different @content_type, so we can then modify our if @trail.save method to handle both photos and gpx, so lets rewrite that as follows

16)

<img src="http://salterhebble.com/blogpics/gxb2.jpg">

and now if we drop into the rails console we should be able to see the trails points, which we can now think about putting onto our trail map

17)

<img src="http://salterhebble.com/blogpics/gxb3.jpg">


We'll need to serialize the points so they appear as part of the trails JSON

18)

<img src="http://salterhebble.com/blogpics/gxb4.jpg">

lets make sure the points appear in the console log

19)

<img src="http://salterhebble.com/blogpics/gxb5.jpg">


and now we can iterate through our points and create a marker out of each, to then be pushed into the array

20)

<img src="http://salterhebble.com/blogpics/gxb6.jpg">

and restart the dokku server as we have new tables in the database

21)

<img src="http://salterhebble.com/blogpics/gxb7.jpg">

but it would also probably be good to change this now to a polyline marking the route, and the ability to add gpx to an existing trail

#Polyline

Lets add the polyline to the google map on the trail page, and wrap it in an ng-if statement, to show when ready

22)

<img src="http://salterhebble.com/blogpics/gpx22.jpg">

we can then create polylineSteps as part of the map object (line 59), and then instead of pushing the Points into the markers array, just push them into the polylineSteps array (line 64/65), and then turn map ready, once completed

23)

<img src="http://salterhebble.com/blogpics/gpx23.jpg">

#Adding GPX to an existing trail

We already have the dropzone posting to the photocontroller, so i decided to reuse this controller to process gpx files, which kind of makes this a file controller rather than a photo controller now, lets have a look at the the photocontroller's create method

24)

<img src="http://salterhebble.com/blogpics/gpx24.jpg">

one the file is received, if its a jpg it can be processed just as before, but now (line 56), if its gpx data, we find the trail, and then update its gpx attribute, with the attached file

25)

<img src="http://salterhebble.com/blogpics/gpxshow.jpg">

#Custom Serializers

If we're going to be serializing the points, we're now increasing the amount of JSON data, potentially quite significantly, and given that a) we only need the points data on the trail view, and not any other views, we should think about reducing the overhead, so to do this i created a custom serializer


26)

<img src="http://salterhebble.com/blogpics/gpx26.jpg">

27)

<img src="http://salterhebble.com/blogpics/gpx25.jpg">

In the original trail serializer, Ive now commented out the associated tracksegments and points models, and created a points serializer, where the points are serialized to form part of the JSON


and now in the index controller, no serializer is specifed, which means it will look for the default trail serializer

28)

<img src="http://salterhebble.com/blogpics/gpx27.jpg">


and in the show method, we can specify the new points serializer, so only on an individual trail page the points will be returned

29)

<img src="http://salterhebble.com/blogpics/gpx28.jpg">

















