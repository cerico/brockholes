---
layout: post
title: "More Angular Controllers"
description: ""
category: 
tags: [angular,javascript,controllers]
summary: "Lets introduce angular controllers for our index, profile, and new pages."
---
Lets introduce angular controllers for our index, profile, and new pages.

#trails index controller

Before we start we'll need to to make root false in our index method for our json data that is serialized, just as we did in our show method


<img src="http://salterhebble.com/blogpics/angticrootfalse.jpg">

Now lets move our TrailController into its own file, and return our app.js.erb back to its original two lines.

<img src="http://salterhebble.com/blogpics/angtic1.jpg">


and now we can create a trailsindex.js.erb for our TrailsIndexController, and output a message to the screen

<img src="http://salterhebble.com/blogpics/angtic2.jpg">

<img src="http://salterhebble.com/blogpics/angtic3.jpg">

<img src="http://salterhebble.com/blogpics/angtic4.jpg">

now lets make a http GET request for the json data, and change our message to output that

<img src="http://salterhebble.com/blogpics/angtic5.jpg">
now we have the trail data we can replace 

<img src="http://salterhebble.com/blogpics/angtic6.jpg">

with 

<img src="http://salterhebble.com/blogpics/angtic7.jpg">

and

<img src="http://salterhebble.com/blogpics/angtic8.jpg">

with  

<img src="http://salterhebble.com/blogpics/angtic9.jpg">

and lets replace our rails counting of bookmarks

<img src="http://salterhebble.com/blogpics/angtic10.jpg">

with angular

<img src="http://salterhebble.com/blogpics/angtic11.jpg">

and our current_users bookmarks can go from 

<img src="http://salterhebble.com/blogpics/angtic12.jpg">

to

<img src="http://salterhebble.com/blogpics/angtic13.jpg">

#Profile Controller

lets create a new Angular ProfileController, and output a basic message to screen to make sure it works


<img src="http://salterhebble.com/blogpics/prof1.jpg">

<img src="http://salterhebble.com/blogpics/prof2.jpg">

<img src="http://salterhebble.com/blogpics/prof3.jpg">

Great, now lets http GET out userdata from the rails server, and output that to the screen

<img src="http://salterhebble.com/blogpics/prof4.jpg">

and if that outputs ok we can start to change out profile page to angular, so

<img src="http://salterhebble.com/blogpics/prof5.jpg">

changes to 

<img src="http://salterhebble.com/blogpics/prof6.jpg">

Now lets generate a user serializer 

<img src="http://salterhebble.com/blogpics/prof7a.jpg">

and move these variables

<img src="http://salterhebble.com/blogpics/prof7.jpg">

into our serializer

<img src="http://salterhebble.com/blogpics/prof8a.jpg">

and we can now reference that in our html, firstly in the sidebar

<img src="http://salterhebble.com/blogpics/prof9.jpg">

Now lets look at our photos div

<img src="http://salterhebble.com/blogpics/prof10.jpg">

In the static app we had jquery conditionally showing the showfaved,showcompleted or showadded divs, lets replace that with angular

<img src="http://salterhebble.com/blogpics/prof11.jpg">

<img src="http://salterhebble.com/blogpics/prof12.jpg">

and lets create an ng-click function (changeBookmarkView )in our side bar menu, to change the bookmarkchoice state to whichever the user clicks on

<img src="http://salterhebble.com/blogpics/prof14.jpg">

and write that function in our profile.js.erb, setting the bookmarkchoice state to the argument it receives from ng-click

<img src="http://salterhebble.com/blogpics/prof15.jpg">

now we can change our trail and trail photos from 
<img src="http://salterhebble.com/blogpics/prof17.jpg">

to

<img src="http://salterhebble.com/blogpics/prof18.jpg">

Now I generated a bookmark serializer here to handle the bookmarks but i ran into some problems with a **stack level too deep** error, so, ive added that as an issue to my trello board to be looked at and will revisit that in a newer post, in the meantime Ive removed the user_serializer and gone with the following solution in the users_controller.rb, to pass the JSON this way

<img src="http://salterhebble.com/blogpics/profr.jpg">

Its a little messy, and horrible to write, but it present the data in the right format, except we no longer have access to the distance calculator that we wrote in the user_serializer, so that will have to be rewritten in the front end, after the http GET request for the JSON user data

<img src="http://salterhebble.com/blogpics/prof22.jpg">

and now we can reference that in our html, limiting the distance travelled to one decimal place

<img src="http://salterhebble.com/blogpics/prof23.jpg">



