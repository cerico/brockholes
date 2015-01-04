---
layout: post
title: "Authentication II: angular,ui-router,devise & oauth"
description: ""
category: 
tags: [angularjs,authentication,ui-router]
summary: "In this post we'll be passing the current_user object to angular, so that ui-router can redirect to a login page for pages that require being logged in. We'll also restrict our dropzone to signed in users, and add twitter and facebook to the list of oauth providers, and add an authenticity_token for our posts/puts"
---

At the moment our authentication is handled by devise and oauth, I wanted this to remain the same, but have angular's ui-router displaying routes according to logged in status and role, redirecting to a login page for views that require a log in (the Add a New Trail view, specifically). So first, we need a splash screen with social sign-ins, which we can redirect to for unauthorized routes. So lets put that here (we still only have oauth for google, so the twitter and facebook links are just placeholders for now)

1) app/assets/javascripts/templates/splash.html

    {% highlight html %}
    <div class="intro">
      <div class="mainloginbox">
        <div class="login loginfirst">
          <a class="splashtitle" href="#">scen.ic</a>
        </div>
      <div class="login">
        <a href="users/auth/google_oauth2"><img class="socialsignin" src="assets/google.png"/></a>
      </div>
      <div class="login">
        <a href="#"><img class="socialsignin" src="assets/twitter.png"/></a>
      </div>
      <div class="login">
        <a href="#"><img class="socialsignin" src="assets/facebook.png"/></a>
      </div>
      <div class="login loginlast">
        <ul>
          <li><a class="splashlinks" href="/#trails">Discover </a></li> 
          <li><a class="splashlinks2" href="/#map">Map</a></li>
        </ul>
      </div>
    </div>
    {% endhighlight %}
    
and setup a controller for the page, at the minute we'll just have it hide the header, and I also put in a randomized background    

2) app/assets/javascripts/controllers/splash.js.erb

{% highlight javascript %}
scenic.controller("SplashController", [ '$scope','$location','$http', function($scope,$location,$http) {
  $(".header").css("visibility","none")
  var backgroundnumber = Math.floor(Math.random() * 7)+1;
     $(".intro").css("background", "url('assets/backdrop" + backgroundnumber + ".jpg') no-repeat center center fixed");

}])
{% endhighlight %}

and we can add a new 'login' state to the ui-router

3) app/assets/javascripts/app.js.erb

{% highlight javascript %}
59  $stateProvider.state('login', {
60    url: '/login',
61    templateUrl: 'login.html',
62    controller: 'LoginController'
63  });
{% endhighlight %}

Now I want to be verify in angular whether the current user is logged in, so lets make a factory that will return if the user is logged in or not, we'll hardcode it first, then when it works we'll pass it the current_user object, so firstly

4) app/assets/javascripts/app.js.erb

{% highlight javascript%}
1 scenic = angular.module('scenic',['controllers','templates','ui.router','uiGmapgoogle-maps'])
2 
3 controllers = angular.module('controllers',[])
4 
5 scenic.factory('Auth',function() { 
6   return { 
7     isLoggedIn : true
8   }; 
9 })
{%endhighlight%}

and lets first look at the new trail controller, we'll add our new Auth factory as a dependency, and then display a message if we're logged in (and we'll be hardcoded as logged in, to begin with)

5) app/assets/javascripts/controllers/new.js.erb

{% highlight javascript %}
1 scenic.controller("NewController", [ '$scope','$location','uiGmapGoogleMapApi','Auth', function($scope,$location,uiGmapGoogleMapApi,Auth) {
2   $scope.auth = Auth;
3   console.log(Auth)
4   if ($scope.auth.isLoggedIn){
5     console.log("im logged in")
6   }
    {%endhighlight%}

6) console.log

<img src="http://salterhebble.com/blogpics/imloggedin.jpg">

Now, lets pass into Angular our rails/devise current_user object

7) app/views/layouts/application.html.erb

{% highlight html%}
         <% if current_user %>
        <script>
          var current_user = <%=raw current_user.to_json  %>
        </script>
      <% end %>
{% endhighlight %}

and in our factory we can now change our isLoggedIn status dependent on the presence of the current_user variable

8) app/assets/javascripts/controllers/new.js.erb

{% highlight javascript %}
5  scenic.factory('Auth',function() { 
6    if (typeof current_user !== 'undefined'){
7      return { 
8        isLoggedIn : true
9      }; 
10   }else{
11     return { 
12       isLoggedIn : false
13     }; 
14   }
15  })
 {%endhighlight%}

Now, lets use this to redirect a non-signed in user from the NewTrail view to the login view (note that the Trails#new controller already blocks trail uploads unless a user is signed in, so this redirection we're doing now isn't for security but for UX, not showing a view that a non-logged in user will be unable to use). Firstly in our state for the newtrail view, lets use our Auth factory's isLoggedIn boolean

9) app/assets/javascripts/app.js.erb

{% highlight javascript%}
48  $stateProvider.state('new', {
49    url: '/new',
50    templateUrl: 'new.html',
51    controller: 'NewController',
52    data : {requireLogin : true },
53  }),
{% endhighlight%}

and in the same file we can now write a run block

10) app/assets/javascripts/app.js.erb

{% highlight javascript %}
69 scenic.run(function ($rootScope, $state, $location, Auth) {
70
71   $rootScope.$on('$stateChangeStart', function (event, toState, toParams, fromState) {
72
73     var shouldLogin = toState.data !== undefined
74       && toState.data.requireLogin 
75       && !Auth.isLoggedIn ;
76
77     if(shouldLogin) {
78      $state.go('login');
79      event.preventDefault();
80      return;
81     }
82   });
{% endhighlight %}

so here I'm setting up a shouldLogin boolean, which becomes true when a ui-router state has data attached to it (which we just did added to thew 'new' state in slide 8), and that data is set to requireLogin true, and the Auth factory's isLoggedIn boolean is false. So this should only affect non-logged in users in the 'new' view.

Then when shouldLogin boolean becomes true, the state gets changed to 'login', and the non-logged in user is redirected to the login page.

We can now also hide the dropzone on the trailpage from logged-out users, by only instantiating the dropzone if the user is logged in. As the dropzone is instantiated as part of a photoupload function, we'll only run that function if the user is logged in, and remove the png from view if logged out.

11) app/assets/javascripts/trail.js.erb


{% highlight javascript %}
1  scenic.controller("TrailsController", ['$scope','$location','$http','uiGmapGoogleMapApi','Auth', function($scope,$location,$http,uiGmapGoogleMapApi,Auth) {
2
3  $scope.auth = Auth



138 if ($scope.auth.isLoggedIn){
139   $scope.photoUpload()
140 }else {
141   $(".dropzone").css("background", "none");
142 }
{% endhighlight %}


Up until now, I've just been letting anonymous users add photos to existing trails, so now we are restricting at the client end, lets also restrict on the server, and require a current_user, and add the currentuser id to the photo

12) app/assets/controllers/photos_controller.rb

{% highlight ruby %}
def create   
  if current_user
    if params[:file].content_type === "image/jpeg"
      @photo = Photo.new(trail_id: params[:trail_id],image: params[:file],user_id: current_user.id)
      {% endhighlight%}
        


#A word about authenticity_token

Firstly lets comment the line which is currently skipping csrf protection in the trails controller 

13) app/controllers/trails_controller.rb


{% highlight ruby %}
1 class TrailsController < ApplicationController
2  # GET /trails
3  # GET /trails.json
4  # skip_before_filter :verify_authenticity_token 
    {% endhighlight %}

And we can now see that POST/PUT actions no longer work as the authenticity_token isnt being passed

14) terminal 

{% highlight bash %}
WARNING: Can't verify CSRF token authenticity
Completed 500 Internal Server Error in 1.1ms
      {% endhighlight %}


So lets add the token in the headers in the dropzone (line 126-128)

15) app/assets/javascripts/controllers/trail.js.erb

{% highlight javascript %}
124 Dropzone.options.photoDropzone = new Dropzone("#media-dropzone", {
125      url: $scope.posthere,
126      headers: {
127        "X-CSRF-Token" : $('meta[name="csrf-token"]').attr('content')
128      },
129      dictDefaultMessage:"",
130    });
131    return Dropzone.options.photoDropzone.on("success", function(file, theNewPhoto) {
132      $scope.trail.photos.push(theNewPhoto)
133      $http.get($location.path() + '/photos.json').success (function (data){
134        Dropzone.options.photoDropzone.removeAllFiles();
135      });
136    })
    {% endhighlight %}
    

Lets also add the token to the default headers, so we can bookmark or complete trails without csrf problems

16) app/assets/javascripts/app.js.erb

{% highlight javascript %}
5 scenic.config(['$stateProvider','$urlRouterProvider','$locationProvider','uiGmapGoogleMapApiProvider','$httpProvider', function ($stateProvider, $urlRouterProvider, $locationProvider, uiGmapGoogleMapApiProvider,provider){
6
7  provider.defaults.headers.common['X-CSRF-Token'] = $('meta[name=csrf-token]').attr('content')
  {% endhighlight %}
    
#Twitter Oauth

I already covered some of the devise and oauth stuff in an earlier post here ([http://scenicblog.salterhebble.com/2014/12/04/signing-in-devise-oauth-and-google/](http://)) so I wont go into setting that up again, just extending it for twitter and facebook

Currently we're only able to sign in with google), so lets add twitter and facebook sign in's too. Lets add the relevant gems and bundle

17) Gemfile

{% highlight ruby %}
gem 'omniauth-twitter'
gem 'omniauth-facebook'
{% endhighlight %}

then we'll register the app at twitter, and take the keys and put them in our .zshrc (or .bashrc)

18) twitter

<img src="http://salterhebble.com/blogpics/twitter1.jpg">

19) ~/.zshrc (or ~/.bashrc)

{% highlight html %}

140 export TWITTER_AUTH_CLIENT_ID=longdayatblakcrocketiweyt
141 export TWITTER_AUTH_CLIENT_SECRET=mynametoubeotwoen


  {% endhighlight %}
  
So the main 3 files we have to look at are the initializers/devise.rb, the omniauth\_callbacks\_controller, and the user model. Lets do twitter first

20) app/controllers/omniauth\_callbacks\_controller.rb

{% highlight ruby%}

def twitter
  @user = User.find_for_twitter(request.env['omniauth.auth'].except("extra"), current_user)

  if @user.persisted?
    flash[:notice] = "Scenic, find somewhere new to go today!"
    sign_in(@user)
    redirect_to trails_path, event: :authentication
  else
    session["devise.twitter_data"] = request.env['omniauth.auth'].except("extra")
    redirect_to new_user_registration_path
  end
end
  {% endhighlight %}
  
21) app/models/user.rb

{% highlight ruby %}
4    devise :database_authenticatable, :registerable, :recoverable, :rememberable, :trackable, :validatable, :omniauthable, omniauth_providers: [:twitter, :facebook, :google_oauth2]

37 def self.find_for_twitter(auth, signed_in_user=nil)
38  twitter_email = auth.info.nickname.downcase + "@twitter.com"
39  if user = signed_in_user || User.find_by_email(twitter_email)
40    user.provider = auth.provider
41    user.uid = auth.uid
42    user.name = auth.info.name
43    user.photo = auth.info.image
44    user.save
45    user
46  else     
47    where(auth.slice(:provider, :uid)).first_or_create do |user|
48      user.provider = auth.provider
49      user.uid = auth.uid
50      user.name = auth.info.name
51      user.photo = auth.info.image
52      user.email = twitter_email
53      user.password = Devise.friendly_token[0,20]
54      #user.skip_confirmation!       
55      end
56    end
57  end
  {% endhighlight %}
  
22) initializers/devise.rb

{% highlight ruby%}
261    config.omniauth :twitter, ENV['TWITTER_AUTH_CLIENT_ID'], ENV['TWITTER_AUTH_CLIENT_SECRET']
{% endhighlight%}

and in the login template

23) app/assets/javascripts/templates/login.html

{%highlight html%}
    <div class="login">
      <a href="users/auth/twitter"><img class="socialsignin" src="assets/twitter.png"/></a>
    </div>
    {%endhighlight%}
    
and dont forget to set the twitter auths on the dokku/heroku machine

24) terminal

{% highlight html%}
root@apps:~#dokku config:set scenic TWITTER_AUTH_CLIENT_ID=itgoeshere
root@apps:~#dokku config:set scenic TWITTER_AUTH_CLIENT_SECRET=andtheotheronehere
  {% endhighlight%}
    
#Facebook Oauth

And now facebook, couple of slight differences in the files below, but first register the app at developers.facebook.com, and then add the site URL (i put in localhost:3000 at first, then went back once it worked to put the dokku address in)


25) http://developers.facebook.com

<img src="http://salterhebble.com/blogpics/facebook1.jpg">

26) app/controllers/omniauth\_callbacks\_controller.rb

{% highlight ruby%}

def facebook
  @user = User.find_for_twitter(request.env['omniauth.auth'].except("extra"), current_user)

  if @user.persisted?
    flash[:notice] = "Scenic, find somewhere new to go today!"
    sign_in(@user)
    redirect_to trails_path, event: :authentication
  else
    session["devise.twitter_data"] = request.env['omniauth.auth'].except("extra")
    redirect_to new_user_registration_path
  end
end
  {% endhighlight %}
  
27) app/models/user.rb

{% highlight ruby%}
  def self.find_for_facebook(auth, signed_in_user=nil)
    facebook_email = auth.info.email
    if user = signed_in_user || User.find_by_email(facebook_email)
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
        user.email = facebook_email
        user.password = Devise.friendly_token[0,20]
        #user.skip_confirmation!
        
      end
    end
  end
      {% endhighlight %}
      
 28) initializers/devise.rb
 
  {% highlight ruby%}
     config.omniauth :facebook, ENV['FACEBOOK_APP_ID'], ENV['FACEBOOK_APP_SECRET']
   {% endhighlight %}
   
Then when pushed to dokku, set the facebook config id's

29) terminal

{% highlight html %}
root@apps:~# dokku config:set scenic FACEBOOK_APP_ID=itsfacebooktehiownetg
root@apps:~# dokku config:set scenic FACEBOOK_APP_SECRET=andthesecretonetgwgwgtoo
{% endhighlight%}

and we'll change the sign-in/out link in the application.html.erb to point to the new login page

30 app/views/layouts/application.html.erb

{% highlight html%}
      <% if user_signed_in? %>
        <li class="toplink"><%= link_to "sign out", destroy_user_session_path, :method => :delete %></li>
      <% else %>     
        <li class="toplink"><a ng-href="/#login">Sign In</a></li>
      <% end %> 
      {% endhighlight %}
      
 Next, we can think about having the sign-in link be at pop up or modal, but we'll address that when we come to other modals, in a later post
 
 





    
 
    
