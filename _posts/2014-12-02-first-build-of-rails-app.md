---
layout: post
title: "First Build of Rails App"
description: ""
category: 
tags: [rails,html,css,layout]
---
Lets build our Rails App! Its tempting when writing after the fact to leave things or forget things, so im including everything no matter how trivial seeming

`➜ rails new pennine -d postgresql`

I'm using rails 3.2 (for the first iteration).

`➜ cd pennine`

`➜ git init`

Creates a git repository for our app, now we can go to github, sign in, click respositories and create new repository. Ive called mine Pennine. Back to command line and

`➜  pennine git:(master) git add .`

`➜  pennine git:(master) git commit -m "first commit"`

`➜  pennine git:(master) git remote add origin git@github.com:cerico/pennine.git`

` ➜  pennine git:(master) git push origin master`

and we're good to go, but leave the master branch, and go to dev

`➜  pennine git:(master) git branch dev`

`➜  pennine git:(master) git checkout dev`

our our terminal prompt is now 

 `pennine git:(dev)`
 
So lets start with Trails, we'll scaffold our Trails, to give use the Models, Views and Controllers. We'll be removing the views later on, when we introduce AngularJS, but we'll need them for now

`➜  pennine git:(dev) rails g scaffold Trail name county postcode description:text lat:float lng:float rating:integer distance:float user_id:integer`

`➜  pennine git:(dev) rake db:create`

`➜  pennine git:(dev) rake db:migrate`

two more things to do, and we can have a look at our first page. Firstly, we need to delete public/index.html, and secondly we need to add the following line to config/routes.rb, so that our trails page is the default page when we browse to our app

`root :to => 'trails#index'`

ok, lets start our rails server and have a look

`➜  pennine git:(dev) rails server`

<img src="http://salterhebble.com/blogpics/newapp1.jpg">

but we dont have any trails yet!, so lets seed our database with some trails to get us started. Ive put 16 in my db/seeds.rb, but lets just show the first 3 here, for brevity

`Trail.delete_all`

`t1 = Trail.create!(name:"Tarn Hows",county:"Cumbria",description:"x",lat:55.1833,lng:-2.5000,rating:8,distance:11.2)`

`  t2 = Trail.create!(name:"Kielder Forest",county:"Northumberland",description:"x",lat:55.1833,lng:-2.5000,postcode:"CA16 6QP",rating:9,distance:17.7)`

` t3 = Trail.create!(name:"Ullswater",county:"Cumbria",description:"x",lat:54.57467,lng:-2.8715,postcode:"bd24 9qw",rating:10,distance:9.7)`

lets, populate out database and have a look in the browser

`➜  pennine git:(dev) rake db:drop`

`➜  pennine git:(dev) rake db:migrate`

`➜  pennine git:(dev) rake db:seed`
 
 <img src="http://salterhebble.com/blogpics/newapp2.jpg">
 
 now we have some trails, lets make our page being to look a bit more like our template page. Lets take our css from out template site and put it into app/assets/stylesheets, and our background image (j14.jpg) and put it into public, then we'll take out header html and paste it into views/layouts/application.html.erb directly about the line `<%= yield %>`
 
 and our site should now look like 
 
 <img src="http://salterhebble.com/blogpics/newapp3.jpg">
 
 Lets paste everything from <div="main">, into app/views/trails/index.html.erb, below whats already there - we'll leave that for now, as we need to combine the two
 
 In our html, we have 9 of the following blocks,
 
    <div class="index-photobox column-32">
      <a href="trail.html"><img src="https://calderscenic.s3.amazonaws.com/uploads/photo/image/2/frontpage_tod1.jpg"></a>
      <div class="everytrail_photofooter">
        <p class="everytrail_photofooter_name"><a href="trail.html">Calderdale Way</a> </p>
        <p class="everytrail_photofooter_county">West Yorkshire</p>
      </div>
    </div>
    
 Lets delete the last 8, and start to work on changing this div, from static to dynamic
 
 first of all lets take the following trails.each do and end ruby tags and wrap them around this div, so it now looks like this 
 
     <% @trails.each do |trail| %>
     <div class="index-photobox column-32">
      <a href="trail.html"><img src="https://calderscenic.s3.amazonaws.com/uploads/photo/image/2/frontpage_tod1.jpg"></a>
      <div class="everytrail_photofooter">
        <p class="everytrail_photofooter_name"><a href="trail.html">Calderdale Way</a> </p>
        <p class="everytrail_photofooter_county">West Yorkshire</p>
      </div>
    </div>
    <% end %>
    
so now we have a div for each of the 16 trails, but the information in each photobox is the same, so lets remove the hard coded trail and county and replace with the trail name and county from the database, so lets change

`Calderdale Way` to `<%= trail.name %>` and `West Yorkshire` to `<%= trail.county %>`
 
now the names and counties reflect each trail in the database, but the links all go to the same page, so lets fix that too, and change

`<a href="trail.html"><%= trail.name %></a>` to `<%= link_to trail.name, trail %>`

and now the links in each trail div will go to the correct trail

But what about the photos, they're still all the same photo

 <img src="http://salterhebble.com/blogpics/newapp4.jpg">
 
 well, we can look at that in the next post
 

  



