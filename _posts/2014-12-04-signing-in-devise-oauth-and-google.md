---
layout: post
title: "Signing In: Devise, Oauth and Google"
description: ""
category: 
tags: [devise,oauth,google]
---
Our app is going to need users to be able to sign in and create content, so lets set up some authentication. We'll have users sign up with google, with twitter and facebook. Lets start with Google. First we'll need to sign into the developers console at [https://console.developers.google.com/project](http://) and create a new project, then go to the consent screen and add in a product name

<img src="http://salterhebble.com/blogpics/g1.jpg">

then to the credentials screen, where you should have something similar to this

<img src="http://salterhebble.com/blogpics/pennins.jpg">

We'll put in out local machine, our server on dokku and also our server at heroku, this will give us a client_id and a client_secret, we'll put those into our .zshrc (or .bashrc) as environment variables, ready for later, like so

    export GOOGLE_AUTH_CLIENT_ID_SCENIC=jsjgwenognweg.apps.googleusercontent.com
    export GOOGLE_AUTH_CLIENT_SECRET_SCENIC=kwehbgwebg
    
also, remember to go to the API tab, and enable google+ API

<img src="http://salterhebble.com/blogpics/googleplus.jpg">
    

#Devise

lets add devise and google oauth to our Gemfile, and then bundle

`gem 'devise'`

`gem 'omniauth-google-oauth2'`

` pennine git:(master) bundle`

`bundle exec rails g devise:install`

which gives us the following instructions

    Some setup you must do manually if you haven't yet:

    1. Ensure you have defined default url options in your environments files. Here
       is an example of default_url_options appropriate for a development environment
       in config/environments/development.rb:

       config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

       In production, :host should be set to the actual host of your application.

    2. Ensure you have defined root_url to *something* in your config/routes.rb.
       For example:

         root to: "home#index"

    3. Ensure you have flash messages in app/views/layouts/application.html.erb.
       For example:

         <p class="notice"><%= notice %></p>
         <p class="alert"><%= alert %></p>

    4. If you are deploying on Heroku with Rails 3.2 only, you may want to set:

         config.assets.initialize_on_precompile = false

       On config/application.rb forcing your application to not access the DB
       or load models when precompiling your assets.

    5. You can copy Devise views (for customization) to your app by running:

         rails g devise:views
         

we've already done steps 2 and 4, so lets do 1, and then 5, and then let devise create a user model for us

`bundle exec rails g devise user`

now, we need to edit the resulting app/models/user.rb file

We need to add the following to Devise list, 

`:omniauthable, omniauth_providers: [:google_oauth2]`

so that devise is now ominauthable, we'll be using multiple providers, but for now its just going to be google, so we just have the one in there

then we'll also need to add in the following method

    def self.find_for_google_oauth2(auth, signed_in_user=nil)
      if user = signed_in_user || User.find_by_email(auth.info.email)
        user.provider = auth.provider
        user.uid = auth.uid
        user.name = auth.info.name
        user.photo = auth.info.image
        user.save
        user

      else
        where(auth.slice(:provider, :uid)).first_or_create do |user|
          user.provider = auth.provider
          user.uid = auth.uid
          user.name = auth.info.name
          user.photo = auth.info.image
          user.email = auth.info.email
          user.password = Devise.friendly_token[0,20]
          # user.skip_confirmation!
        end
      end
    end


Now lets modify our layouts/application.html.erb, which currently staticly says "sign out"!

so lets take out 

`a href="index.html">Sign out</a></li>`

and replace with the following block

    <% if user_signed_in? %>
      <li>
        <%= link_to "sign out", destroy_user_session_path, :method => :delete %>
      </li>

    <% else %>
      
      <li>
        <%= link_to "sign in", user_omniauth_authorize_path(:google_oauth2)%>
      </li>
 
    <% end %>  
    
Now let generate an omniauth callbacks controller, 

`rails g controller omniauth_callbacks`

and in the resulting app/controllers/omniauth_callbacks_controller.rb file, lets add the following method

      def google_oauth2
        @user = User.find_for_google_oauth2(request.env['omniauth.auth'], current_user)
        if @user.persisted?
          flash[:notice] = "Scenic, find somewhere new to go today!"
          sign_in(@user)
          redirect_to trails_path, event: :authentication
        else
          session["devise.google_data"] = request.env['omniauth.auth']
          redirect_to new_user_registration_path
        end
     end


We'll need to edit our devise_for :users line, in config/routes.rb to the following, to reflect our ominauth controller

`
  devise_for :users, controllers: { omniauth_callbacks: "omniauth_callbacks" }`

Now we'll need to add in out google credentials from earlier, which are in our .zshrc, so we'll add the following into our config/initializers/devise.rb

      config.omniauth :google_oauth2, ENV['GOOGLE_AUTH_CLIENT_ID']  ENV['GOOGLE_AUTH_CLIENT_SECRET'], scope: "email, profile", client_options:{ image_aspect_ratio: "square", image_size: 30 }
      
the environment variables must be stated exactly as above, i tried with app-named env variables, it worked locally but caused problems on dokku and heroku


we dont directly paste in our google keys (or amazon or twitter), as then they'll be visible from the github page, so this just references them as environment variables from the zshrc

Now lets add the following to our user model, with this migration

`rails g migration AddProviderAndUidAndNameAndPhotoToUsers provider uid name photo`

`rake db:migrate`

#Dokku


I had problems initially with differently named google_auth env variables, but once i changed back to default, worked ok, so lets just push to our droplet

`➜  pennine git:(master) git push dokku master`

and then from our droplet, set up the config variables

    dokku config:set scenic GOOGLE_AUTH_CLIENT_ID=blahjblah.apps.googleusercontent.com
    dokku config:set scenic GOOGLE_AUTH_CLIENT_SECRET=wiouguowebgw
    
and we should be good to go

#heroku

As above, problems introduced by using non-standard naming scheme for ENV variables, but once that was rectified, pushing to heroku was as simple as the following

    heroku config:set GOOGLE_AUTH_CLIENT_SECRET=ouwegbouwebgw
    heroku config:set GOOGLE_AUTH_CLIENT_ID=erherherhe.apps.googleusercontent.com
    heroku run rake db:migrate
    heroku restart
    
 so now we have signing in via google oauth, we'll need to add to the trello board for twitter and facebook signins, and to use the front page we created in our dummy app..



       









