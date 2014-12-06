---
layout: post
title: "Profile Page (Initial)"
description: ""
category: 
tags: []
---
A short post here as we still need a profile page for our users.

Lets create a controller for our users, and we'll just need a show page, at least for the moment

`rails g controller Users show`

and have a simple show method in there

    def show
      @user = User.find(params[:id])

      respond_to do |format|
        format.html # show.html.erb
        format.json { render json: @photo }
      end
    end
    
and add `resources :users` into our config/routes.rb, and we should be able to browse to users/1 to see our first user


Lets change our hardcoded profile link in the application page from 

    <a href="map.html">Map</a>`

to 

    <% if user_signed_in? %>   
      <li><%= link_to current_user.name.split[0], current_user%></li>
    <% end %> 
    



and now can paste in our html from the profile page in the dummy app into the resulting views/show.html.erb page, from our main div to the final div. We cant change a lot on this page yet, but lets change our user image from

`<img class="profilephoto" src="gareth.jpg"></img>`

to 

`<%= image_tag(@user.photo, :class => "profilephoto") %>`

and our users hardcoded name to `<%= @user.name %>`

and then we'll need to get rid of a bunch of the hardcoded divs for each trail, so we should have something like this 

    <div class="profile_trailphotos column-100 showfaved">

      <div class="userbit column-100">Favourited</div>
      <div class="eachphoto column-32"><a href="trail.html"><img src="/assets/eachphoto1.jpg"></a>
        <div class="little_photofooter">
          <span class="little_photofooter_left" >Calderdale Way</span>   
          <div class="little_photofooter_right">
            <ul>
              <li><i class="fa fa-heart-o"></i></li>
              <li><i class="fa fa-check circle"></i></li>
            </ul>
          </div> <!-- close of footerright-->
        </div><!-- close of photofooter-->
      </div><!-- close of eachphoto -->
        
    </div><!-- close of profile_trailphotos-->      
        

    <div class="profile_trailphotos column-100 showcompleted">
      <div class="userbit column-100">Trails completed by Gareth</div>
        <div class="eachphoto column-32"><a href="trail.html"><img src="/assets/eachphoto1.jpg"></a>
        <div class="little_photofooter">
          <span class="little_photofooter_left" >Calderdale Way</span>   
          <div class="little_photofooter_right">
            <ul>
              <li><i class="fa fa-heart-o"></i></li>
              <li><i class="fa fa-check circle"></i></li>
            </ul>
          </div> <!-- close of footerright-->
        </div><!-- close of photofooter-->
      </div>

    </div><!-- close of profile_trailphotos-->

    <div class="profile_trailphotos column-100 showadded">
      <div class="userbit column-100">Trails added by Gareth</div>
      <div class="eachphoto column-32"><a href="trail.html"><img src="/assets/eachphoto1.jpg"></a>
        <div class="little_photofooter">
          <span class="little_photofooter_left" >Calderdale Way</span>   
          <div class="little_photofooter_right">
            <ul>
              <li><i class="fa fa-heart-o"></i></li>
              <li><i class="fa fa-check circle"></i></li>
            </ul>
          </div> <!-- close of footerright-->
        </div><!-- close of photofooter-->
      </div>

    </div><!-- close of profile_trailphotos-->
    
We'll now need to set up our Bookmarking controller and model, in order to present this data, but thats a story for another day

 

