---
layout: post
title: "Dropzone II"
description: ""
category: 
tags: [dropzone,jquery,angular]
summary: "Combining the dropzone plugin with a regular form to upload trails and associated images simultaneously"
---
Now lets also make a dropzone uploader for our newtrails template. There is a subtle difference here. On the existing trail page, the photo is just uploaded onto the page/trail, and belongs_to that trail - and it goes there as soon as it is dropped on the page. On the new trail page, there is no trail for it to belongs_to...yet, so as soon as its added, it has to be held back, and then submitted as part of a regular form

First lets add a bit of css to give the dropzone a background of the photoupload.png, so it looks like this 

<img src="http://salterhebble.com/blogpics/tdrop1.jpg">

With the more recent versions of the dropzone plugin, the following happens when the form is uploaded

<img src="http://salterhebble.com/blogpics/params1.jpg">

and earlier version presented the files in this format, which is nicer to iterate through

<img src="http://salterhebble.com/blogpics/params2.jpg">

I wanted to use version 3.8.7, so i set up the bowerfile with that version, installed and required as follows

<img src="http://salterhebble.com/blogpics/dropbower1.jpg">
<img src="http://salterhebble.com/blogpics/dropbower3.jpg">
<img src="http://salterhebble.com/blogpics/dropbower2.jpg">

and while it appears to install correctly, it was actually using the most recent version instead, so i removed the line from application.js, and simply downloaded version 3.8.7 directly into app/assets/javascripts/dropzone.js.erb, at least until I can make the newer version work the way I'd like. So here is my dropzone configuration in the newtrail.js.erb template, lets run through it

<img src="http://salterhebble.com/blogpics/newjs1.jpg">
<img src="http://salterhebble.com/blogpics/newjs2.jpg">

As before Line 5 prevents autodiscovery  of a dropzone as we're defining it manually, and the dropzone configuration itself is lines 8-57. I'm adding in lines 10 and 11, to hold back uploading of the images, so they can be submitted as part of the form along with the trail data, then I have an event listener on  the submit button, which has a preventDefault on it (otherwise the form could be submitted without images, then if the number of files submitted is less than 1, line 28 shows the picuploaderror div (below). If successfully uploaded to the server, we should get a response with the new trail id (line 44), and line 47 will redirect us to the newly created trail. Line 62 created the dropzone, on the element with the id "trail-dropzone"

<img src="http://salterhebble.com/blogpics/picuploaderror.jpg">

We need to make modifications to a rails trail controller for this to work (shown below)

<img src="http://salterhebble.com/blogpics/trailcreate.jpg">

Here we can see the defauilt line 54, has been replaced, based on the params hash below. Then we'll iterate through each of the files receoved, created a new Photo from it, and push it into the newly created @trail.photos array. As we're using Active Model Serializer we'll also have to edit line 65 to as shown, otherwise redirection to the newly created trail can't occur

<img src="http://salterhebble.com/blogpics/params2.jpg">

We still need to take care of the trail co-ordinates and uploader, but we'll leave those hardcoded for now, as we'll need to add angular-google-maps here, which can be in the next post

