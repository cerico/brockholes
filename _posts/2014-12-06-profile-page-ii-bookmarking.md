---
layout: post
title: "Profile Page II: Bookmarking"
description: ""
category: 
tags: [javascript,angularjs,bower]
---
Now our user can sign in, and they have a profile page, but now they need to be able to save trails as favourites, and also mark trails as completed, and have this show on their profile page.

Firstly we'll need a bookmarks controller, because a user can have many bookmarks, and a trail can also have many bookmarks, so the bookmarks controller can work here, and we'll have booleans for completed and for favourited

Lets generate a model and a controller

    ➜  pennine git:(bookmarking) rails g scaffold Bookmark trail_id:integer user_id:integer completed:boolean favourited:boolean rating:integer

    
    
so, our bookmark model, should look like this 

    class Bookmark < ActiveRecord::Base
      attr_accessible :completed, :favourited, :rating, :trail_id, :user_id

      belongs_to :trail
      belongs_to :user
    end
    
 and make sure that both our trail model and our user model, both have 
 
 `  has_many :bookmarks`
   
Lets go to our trail#show page, and just put some additional information somewhere on the screen, right at the top of the page as this is just temporary. We'll show our currently logged in user, and we'll all the bookmarks the trail has, and we'll show all the bookmarks our currently logged in user has 

      <h1 class="bright">you are <%= current_user.name %><hr></h1>
      <h1 class="bright">your bookmarks are <%= current_user.bookmarks %><hr></h1>
      <h1 class="bright">this trails bookmarks are <%= @trail.bookmarks %><hr></h1>
  
      
except, we havent got any bookmarks yet, lets create a couple from the console, lets run `rails c`, and then 

    [2] pry(main)> Bookmark.create(user_id:1,trail_id:5,favourited:true)
    [2] pry(main)> Bookmark.create(user_id:1,trail_id:6,completed:true)
    
 now if we go to our page, we can see something like this
 
 <img src="http://salterhebble.com/blogpics/bookmarks1.jpg">
 
In the trail controller, lets do the following to find the current users bookmarks, for this trail (which should only return one record)
 
     @bookmark = current_user.bookmarks.find_by_trail_id(@trail.id)
     
 and in the html, we can now show that bookmark, for both a favourited trail, and a completed trail, from the same record
 
    favourited <%= @bookmark.favourited  if @bookmark.favourited?%><hr>
    completed <%= @bookmark.completed  if @bookmark.completed?%><hr>
    
and lets go back to the controller, and also return the total number of finishers and favourites for the trail 

    @bookmark = current_user.bookmarks.find_by_trail_id(@trail.id)
    @favourites = @trail.bookmarks.where(favourited:true)
    @completeds = @trail.bookmarks.where(completed:true)
    
    
so, now in our html we have 

    did you fav this one? <%= @bookmark.favourited  if @bookmark.favourited?%><hr>
    did you complete this one? <%= @bookmark.completed  if @bookmark.completed?%><hr>
    all favourited <%= @favourites.length %><hr>
    all completed <%= @completeds.length %>
 
 and that looks someting like this
 
 <img src="http://salterhebble.com/blogpics/bookmark2.jpg">
 
 so, now we can start change our font-awesome hearts and ticks to reflect this data, changing the '12' to `<%= @completeds.length %>` and the '21' to `<%= @favourites.length %>` , but we;ll also need to wrap that in a block in case there are no completed or favourites for this particular trail
 
    <% if @completeds.present? %>
      <li><a href="#"><%= @completeds.length %>  Hikers <i class="fa fa-check circle green"></i></a></li>
    <%end%>
    <% if @favourites.present? %>
      <li><a href="#"><%= @favourites.length %> Hikers <i class="fa fa-heart red"></i></a></li>
     <% end %>
 
 and then the heart and tick we inside the photo, we can wrap in an if statement, so we can have something like this 
 
    <ul>
      <% if @bookmark.present? && @bookmark.favourited? %>
        <li><i class="fa fa-heart red"></i></li>
      <% else %>
        <li><i class="fa fa-heart-o"></i></li>
      <% end %>
      <% if @bookmark.present? && @bookmark.completed? %>
        <li><i class="fa fa-check circle green"></i></li>
      <% else %>
        <li><i class="fa fa-check circle"></i></li>
      <% end %>
    </ul>
    
    
Lets do similar to the above, in our users controllers lets add in

    @user = User.find(params[:id])
    @favs = @user.bookmarks.where(favourited:true)
    @completed = @user.bookmarks.where(completed:true)
    
    
 and we also have access to the trails added data via 

    @added = Trail.where(user_id:@user.id)
    
so lets add that in too, and we can changed the hardcoded user numbers on the right. lets also change our total distance covered, so in the user controller we can also add in

    `@distance = view_context.distance_calc(@completed)`
    
which defines a method "distance calc", which is going to take the argument of @completed - ie, our users completed trails. Now we'll need to write the distance_calc method, and put in the application_controller.rb. Here is mine, and right now its the only method in there, so i'll show everything here

    class ApplicationController < ActionController::Base
      protect_from_forgery


      helper_method :distance_calc

      def distance_calc(completedhikes)
        totaldistance = 0
        completedhikes.each do |hike|
          totaldistance += hike.trail.distance
        end
        totaldistance
      end
    end
    
and that can go straight into our profile page as `Hiked: <%= @distance %> Miles`

So now we are able to see bookmarked data on both the trails#show page, and the profile page, we can also do something in the trails#index page, with a block like this to show how many users have favourite or completed a trail, and also whether we have completed or favourited a trail (lets also add a little bit of css in, while we are here, so our 

    <div class="everytrail_photofooter_userinfo">
      <%= trail.bookmarks.where(completed:true).length if trail.bookmarks.present? %>
        <%= trail.bookmarks.where(favourited:true).length %>
    </div>
                          
    <div class="everytrail_photofooter_userinfo">
      <%= trail.bookmarks.find_by_user_id(current_user.id).completed if trail.bookmarks.present? %>
      <%= trail.bookmarks.find_by_user_id(current_user.id).favourited if trail.bookmarks.present? %>
    </div>
    
which gives us something like this 

<img src="http://salterhebble.com/blogpics/bookmarks3.jpg">

I didnt have any css for this in the dummmy site, so lets quickly do someething now to reflect this. Lets put in our coloured font-awesome heart and tick for if a trail has been completed or favourited, and a smaller white heart and tick for the number of favouriters and completeds, and make the colour of the number white also, lets have something like

    <div class="everytrail">
      <% @trails.each do |trail| %>
        <% if trail.photos.first %>
        <div class="index-photobox column-32">

        <%= link_to image_tag(trail.photos.first.image.frontpage.url), trail %>
          <div class="everytrail_photofooter">
            <p class="everytrail_photofooter_name"><%= link_to trail.name, trail %> </p>
            <p class="everytrail_photofooter_county"><%= trail.county %></p>
            <div class="everytrail_photofooter_userinfo">
              <ul>
                <li><i class="fa fa-check circle"></i><%= trail.bookmarks.where(completed:true).length %></li>
                <li><i class="fa fa-heart-o"></i><%= trail.bookmarks.where(favourited:true).length %></li>
              </ul>
            </div>
            <div class="everytrail_photofooter_userinfo">
              <ul>
                <% if trail.bookmarks.present? && trail.bookmarks.find_by_user_id(current_user.id).favourited %>
                  <li><i class="fa fa-heart red"></i></li>
                <% end %>
                <% if trail.bookmarks.present? && trail.bookmarks.find_by_user_id(current_user.id).completed %>
                  <li><i class="fa fa-check circle green"></i></li>
                <% end %>
              </ul>
            </div>
          </div>
        </div>
        <% end %>
      <% end %>
      
    </div>
      
    .everytrail_photofooter_userinfo {
    float:right;
    color: #f5f6f1;
    font-weight:100;
    margin-top:-2%;
    }
    
This is all well and good, but while we can see these bookmarks, our users arent currently able to create or modify bookmarks, so lets look at that next...



 

 
 
 
 
    