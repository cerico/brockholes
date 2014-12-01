---
layout: post
title: "initial site template"
description: ""
category: 
tags: [html,css,jquery,design,planning,layout]
---
So lets talk about the initial site layout. Rather than wireframing, I made a flat, or dummy site, as ideas about functionality seem easier to visualize with an actual site rather than a wireframe, to me at least, and this can really double up as something of a user journey.

 I have 6 main screens to work with. An index/login page, a trails index page, an individual trail page, a profile page, an add new trail page and a map/search page

Ive kept this mostly as pure html/css, but have added a couple of touches of jquery in here and there, which i will document.

The index page is first 

<a href="http://salterhebble.com/s3/index.html"><img src="http://salterhebble.com/s3/st1.jpg"></a>

<a href="http://salterhebble.com/s3/index.html">http://salterhebble.com/s3/index.html</a>

Very simple log in page, just the ability to log in with any of 3 social media sign ins, probably add in direct signins too at some point. The background is randomized with one of 7 images, with the following bit of jquery

     var backgroundnumber = Math.floor(Math.random() * 7)+1;

     $(".intro").css("background", "url('backdrop" + backgroundnumber + ".jpg') no-repeat center center fixed");
   
   
Logging in then takes to the trail index page 

<a href="http://salterhebble.com/s3/trailsindex.html"><img src="http://salterhebble.com/s3/st2.jpg"></a>

<a href="http://salterhebble.com/s3/trailsindex.html">http://salterhebble.com/s3/trailsindex.html</a>

This just shows us all the trails that are currently on the site, mainly a visual page, the pictures are really doing most of the work here! Clicking on any photo takes us to its individual page


 <a href="http://salterhebble.com/s3/trail.html"><img src="http://salterhebble.com/s3/st3.jpg"></a>
 
 <a href="http://salterhebble.com/s3/trail.html">http://salterhebble.com/s3/trail.html</a>
 
 Here the black sidebar pops out a bit with various bits of information. In the sidebar we have the distance of the trail (which will come from gpx import), the ave rating from each user and its ranking. We can also see how many users have completed the trail, and also how many have favourited it. Moving over to the main content box, we can see which user added this hike to the site, and we have the ability to favourite this trail ourselves, and also mark it as completed. Then we have the map, which will have the route marked out via gpx. Then we have some additional photos, and an area for users to upload their own photos to the trail
 
If we then click on the photo of the user who uploaded the trail, we'll see their profile

<a href="http://salterhebble.com/s3/profile.html"><img src="http://salterhebble.com/s3/st3.jpg"></a>

<a href="http://salterhebble.com/s3/profile.html">http://salterhebble.com/s3/profile.html </a>

Similar to the individual trail page, the sidebar stays out with similar information. How many trails this user has added, how many they have completed. If we are this user, we can also see how many we have favourited, clicking on each of these will switch the main content box between added, completed and favourited. At the moment i've done this with a little bit of jquery, but may switch to angular when we get to that point.

    $(".seefaved").click(function(){
      $(".showfaved").css("display","block")
      $(".showcompleted").css("display","none")
      $(".showadded").css("display","none")
      $(".seefaved a").addClass("current")
      $(".seecompleted a").removeClass("current")
      $(".seeadded a").removeClass("current")

    })
    $(".seecompleted").click(function(){
      $(".showfaved").css("display","none")
      $(".showcompleted").css("display","block")
      $(".showadded").css("display","none")
      $(".seefaved a").removeClass("current")
      $(".seecompleted a").addClass("current")
      $(".seeadded a").removeClass("current")
    

    })
    $(".seeadded").click(function(){
      $(".showfaved").css("display","none")
      $(".showcompleted").css("display","none")
      $(".showadded").css("display","block")
      $(".seefaved a").removeClass("current")
      $(".seecompleted a").removeClass("current")
      $(".seeadded a").addClass("current")
 
    }) 
    
Next we have the 'add new trail' page, here

<a href="http://salterhebble.com/s3/newtrail.html"><img src="http://salterhebble.com/s3/st4.jpg"></a>

<a href="http://salterhebble.com/s3/newtrail.html">http://salterhebble.com/s3/newtrail.html</a>

Here a user will be able to upload a new trail to the site, giving it a name, county and rating, and adding any photos before submitting. At least 1 one photo will have to be submitted or it will mess up the front page, and also who wants to look at a trail with no photos? They should also be able to click on the map to pinpoint its location (and later we'll also add the ability to import gpx data). Also we can see the sidebar pops back in as no longer used.

Finally we have the map page, pretty similar to the add new trail page, except the map now takes up the whole of the div, it should display markers showing the locations of all the trails, and the number of trails on the current map, wherever the user is zoomed in to. The user will also be able to search via the map

<a href="http://salterhebble.com/s3/map.html">http://salterhebble.com/s3/map.html</a>

Now, i also wanted to talk a little bit about responsive design. I havent done a huge amount here, but one thing i wanted to mention was the switching of the header menu. So, below 650px, the header bar changes to display:none and the burger bar menu on the left is displayed, i then have the following bit of jquery to display the site menu when the burger is clicked.

    $(".btn-navbar").click(function(){
     $(".phonemenu").toggleClass("inactive")
     $(".btn-navbar").toggleClass("hamActive")
    })
    
 So, here is the template site (<a href="http://salterhebble.com/s3/map.html">http://salterhebble.com/s3</a>), and also I made an alternative template site here (<a href="http://salterhebble.com/scenicalt/index.html">http://salterhebble.com/scenicalt</a>
), though there are quite a few similarites. As we'll be building alternate versions of the app, its nice to have different sites to look at sometimes so we dont get confused. I thnk its also a good idea not to get married to one version of something, which can easily happen.

Next up we'll make our model diagram, and start to populate the trello board