---
layout: post
title: "nested forms, show page, new page"
description: ""
category: 
tags: [nested_forms,html,css,rails]
---

#show page

Lets get some of the other pages into view, we'll do the show page first, but before that lets just fix our trails index page in the application.html.erb

lets change


to

`<a class="active" href="trailsindex.html">Trails</a>`

to 

`<%= link_to "trails", trails_path, :class => "active" %> `

Now lets paste out trail.html from the dummy app into our trails/show.html.erb and replace all our hard coded data, eg replace

`Calderdale Way` with `<%= @trail.name %>`

our main photo should change from

`<img class="trailphoto column-100" src="960x480.jpg">` 

to

`<%= image_tag(@trail.photos.first.image.trailpage, :class => "trailphoto column-100") %>`

and we can replace this whole block

              <div class="extraphoto column-25"><img src="eachphoto0.jpg"></a>
              </div>
              <div class="extraphoto column-25"><img src="eachphoto1.jpg"></a>
              </div>
              <div class="extraphoto column-25"><img src="eachphoto2.jpg"></a>
              </div>
              <div class="extraphoto column-25"><img src="eachphoto3.jpg"></a>
              </div>
              <div class="extraphoto column-25"><img src="eachphoto3.jpg"></a>
              </div>
              
with

              <% @trail.photos.each do |photo| %>
                <div class="extraphoto column-25"><%= image_tag(photo.image.detail) %></a>
                </div>
              <% end %>
              
we'll have to leave the trail uploader hardcoded for now

#trails#new page

in the application html erb, lets change

`<a href="newtrail.html">Add New</a>` to `<%= link_to "new", new_trail_path %>` and paste out newtrail.html from the template app, into the _form.html.erb

we'll convert our form from 

` <form id="trail-dropzone" class="newtrailform">`

to

`   <%= form_for @trail, :html => { :class => "newtrailform", :id => "trail-dropzone",  multipart: true }  do |f| %>`

and copy the photoupload.png that forms the background, into app/assets/images

and with our form fields, we'll replace all our fields as follows

`<input placeholder="Trail Name" type="name" name="name" >`

with eg

` <%= f.text_field :name, :placeholder => "Trail Name" %>`

and our number block, here

             <select class="rating1" name="rating">
                    
                    <ul class="ratingoptions">
                      <option value="">Rating</option>
                      <option value="1">1</option>
                      <option value="2">2</option>
                      <option value="3">3</option>
                      <option value="4">4</option>
                      <option value="5">5</option>
                      <option value="6">6</option>
                      <option value="7">7</option>
                      <option value="8">8</option>
                      <option value="9">9</option>
                      <option value="10">10</option>
                    </ul>
                  </select>
                  
 with
 
 ` <%= f.select(:rating, options_for_select(1..10 ), {}, {:class => 'rating1'})  %>`
 
 Lets just go back to the main page a second, and wrap each trail div in an if statement, to only show a trail if it has an image present
 
    <% @trails.each do |trail| %>
      <% if trail.photos.first %>
        <div class="index-photobox column-32">
        <%= link_to image_tag(trail.photos.first.image.frontpage.url), trail %>
          <div class="everytrail_photofooter">
            <p class="everytrail_photofooter_name"><%= link_to trail.name, trail %> </p>
            <p class="everytrail_photofooter_county"><%= trail.county %></p>
          </div>
        </div>
      <% end %>
    <% end %>
    
 and similar on the trails#show page with ` <% if @trail.photos.first %>`
 
 Returning to our trail#new page, we're going to need some hidden fields
 
        <%= f.hidden_field :user_id, :value => current_user.id %> 
        <%= f.hidden_field :distance, :value => "12.6" %>
        <%= f.hidden_field :lat, :value => "55.1" %>
        <%= f.hidden_field :lng, :value => "-2.1"%>
        
        
The first shows the user that just uploaded the trail, and the other 3 are temporary, the miles won't be input by a user, that will be calcuated later on via gpx import, and the initial latlng will be input by clicking on the map, but thats also for a little later. Now we need to look at photouploading, but before we do that, now that we dynamically have the trail uploaders id, lets put that onto the trails#show page, and replace

Firstly, we'll need to modify our trails controller, to show the user, so lets add the following @uploader line in, like so

    def show
      @trail = Trail.find(params[:id])
      @uploader = User.find_by_id(@trail.user_id)

and now we can replace

`<img class="userphoto" src="gareth.jpg">`

with

`<%= image_tag(@uploader.photo, :class => "userphoto") %>`

of course our seed data doesn't have an uploader, so lets add `user_id:1` to our db/seeds.db so that our seed data has an uploader of our first user (ie, me) and reseed our database (`rake db:seed`)

We'll also want to restrict out trail uploading to signed in users, so lets change our trails#new.html.erb to the following

    <%if current_user %>
      <%= render 'form' %>

    <% else %>

      lets put our sign in stuff here, later, for signed out users

    <% end %>


#photos

Now our new trail has_many photos, so we'll use the nested_form gem so that when a user adds a photo with their new trail, that photo belongs_to the right trail, so lets add

`gem "nested_form" to our Gemfile, and bundle

and in our models/trail.rb, we'll need to add

`  accepts_nested_attributes_for :photos, allow_destroy: true`

and for our images in our form, we can add in the following so it passes an array of images

`<%= file_field_tag "file[]", type: :file, multiple: true %>`

and in our trail controllers, create method, we'll need something like this, which for each file uploaded, will create a new Photo and make sure that it belongs_to the newly created trail

    if @trail.save
      params[:file].each do |photo|
       @trail.photos << Photo.create(image: photo)   
    end


We won't style the 'choose files' button right now, as we'll be replacing this with a drag and drop area in a little while





 
 
 

