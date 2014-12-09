---
layout: post
title: "Bookmarking II: AngularJS & Active Model Serializer"
description: ""
category: 
tags: [angularjs,bower,active model serializer,javascript]
---

#Angular

Currently we can show existing bookmarks for a user, except the user is unable to actually create these bookmarks in the first place, but we also want the page to reflect the favourite or completed status for the user immediately, and not just on refresh, and that requires Javascript, and we're going to be using a front-end framework to do that, so we're going to need to include Angular in our app, and this means we're going to have a lot of front-end assets in our app, which we're going to be managing with Bower, which we'll install with Node Package Manager....which, if not present already, is installed with Brew

    ➜  pennine git:(bookmarks) ✗ brew install node
    ➜  pennine git:(bookmarks) ✗ npm install bower
    
and to connect rails to our front end assets, we'll use the bower-rails gem, and while we're here lets add in a few more gems for each of deployment to heroku/dokku

    gem 'bower-rails'
    gem "foreman"
    group :production, :staging do
      gem "rails_12factor"
      gem "rails_stdout_logging"
      gem "rails_serve_static_assets"
    end
    
Just the same as with our Gemfile for ruby gems, we'll now have a Bowerfile, so lets create that in the root of our app, and add angular

      asset 'angular'

and just as we bundle, with rails, we can now do the equivalent to install angular

     ➜  pennine git:(bookmarks) ✗ bundle exec rake bower:install
     
and lets check everything is installed ok

    ➜  pennine git:(bookmarking) ✗ ls vendor/assets/bower_components/angular
    README.md           angular.js          angular.min.js.gzip bower.json
    angular-csp.css     angular.min.js      angular.min.js.map  package.json
    
 i had heroku/dokku problems here, so you may need to add both angular.min.js.gzip and angular.min.js.map to your `.gitignore`
 

Now lets require angularjs in our app. In app/assets/javascripts/application.js, lets add in angular so we have something like 

    //= require jquery
    //= require jquery_ujs
    //= require angular/angular
    //= require_tree .
    
 
and we'll reference out angular app right at the top of our layouts/application.html.erb

    <body ng-app="scenic">
    
and lets create our scenic angular app, in app/assets/javascripts/app.js.erb, an

    scenic = angular.module('scenic',[])
        
Now we'll need a trail controller to look after the trails page, so lets add in controllers to our scenic app, and define a controller, our scenic app now changes to


    scenic = angular.module('scenic',['controllers'])
    
and we can add in a controller - and give it something to output on the page

    controllers = angular.module('controllers',[])

    scenic.controller("TrailsController", [ '$scope', function($scope) {
    $scope.message = "hi dere im the trails controller"
    }])
    
 and on our trails page lets reference our trails controller in the main div, and make sure we can see our message
 
     <div class="main" ng-controller="TrailsController">
     
     {{message}}
     
<img src="http://salterhebble.com/blogpics/ang1.jpg">
   
 The first two things our trails controller is going to need is $location, so it knows which page we're on, and $http so it can send and receive data to the rails server. Lets add that in, and change our message to output the url (later on we'll move to having angular handle the views, but at the moment they're still rails views)
 
        scenic.controller("TrailsController", [ '$scope','$location','$http' function($scope,$location,$http) {
     $scope.message = $location.$$absUrl
    }])
    
 and our page should now look something like this 
 
<img src="http://salterhebble.com/blogpics/ang2.jpg">
 
 Now lets have angular make a GET request and put the trail data on the page, lets remove out original $scope.message, and have the results of the GET request become that message
 
     $http.get($location.$$absUrl + '.json').success (function (data){
     $scope.message = data
     })
     
 and now our page should look like this, 
 
 <img src="http://salterhebble.com/blogpics/ang3.jpg">
 
 great! our rails trails controller is returning json data about the trail. but what about the bookmarks? Being an associated model, bookmarks are not returned as part of our json data, to do this we're going to need Active Model Serializer
 
#Active Model Serializer


Lets add this into our Gemfile, and bundle

    gem 'active_model_serializers'
    
and then generate our serializer for the trails model

    rails g serializer trail
    
Lets look at our newly created app/serializers/trail_serializer.rb

    class TrailSerializer < ActiveModel::Serializer
      attributes :id
    end
    
Not much there! lets look in our page

<img src="http://salterhebble.com/blogpics/ams1.jpg">

all our previous data has now gone, and been replaced with just whats in the trail serializer, so lets add back in what we want to see that we had before

    class TrailSerializer < ActiveModel::Serializer
      attributes :id, :name, :county, :rating, :user_id, :distance, :lat, :lng


    end

But what active model serializers gives us is, the ability to add more data to our json, including associated models, so as a trail has_many bookmarks, we can pass that along as part of our json data, so lets do that 

    class TrailSerializer < ActiveModel::Serializer
      attributes :id, :name, :county, :rating, :user_id, :distance, :lat, :lng

      has_many :bookmarks
    end

 
 
 <img src="http://salterhebble.com/blogpics/ams2.jpg">
 
 and now we can see that our json data has also returned an associated bookmarks hash, which shows there is one bookmark, from user_id 1, who has favourite but not completed. Great, so now we can see the total number of bookmarks this trail has, but whatabout the bookmark status for our logged in user? As we need to be able to show the currently logged_in users bookmark condition. Lets go back to our serializer, and add in the following method
 
    class TrailSerializer < ActiveModel::Serializer
      attributes :id, :name, :county, :rating, :user_id, :distance, :lat, :lng, :userbookmark

      has_many :bookmarks
      
      def userbookmark
        scope.bookmarks.find_by_trail_id(object) if scope
      end

      
    end
    
we've added another attribute (userbookmark), which is a method which finds bookmarks for the current trail (object), for the current user (scope), if that scope exists (the default scope here is current_user), and we can also pass current_user along here as well

    
    class TrailSerializer < ActiveModel::Serializer
      attributes :id, :name, :county, :rating, :user_id, :distance, :lat, :lng, :userbookmark, :user

      has_many :bookmarks

      def userbookmark
        scope.bookmarks.find_by_trail_id(object) if scope
      end

      def user
        scope
      end
    end
    
this means we can pass out current_user object directly into angular, without using the gon gem

now lets have a look at our page

<img src="http://salterhebble.com/blogpics/ams3.jpg">

lot more data! Lets change one final thing, in the trail controller lets append root:  false to our show methods json output

          format.json { render json: @trail, root: false  }
          
 this means that trail wont come inside a trail hash but will be at the top level, eg name, not trail.name (this may seem counterintuitive, but when we come to the javascript we'll be referencing these attributes as part of $scope.trail, so this means we'll get $scope.trail.name instead of $scope.trail.trail.name)

ok! So now we have all our info via json, including the associated bookmark info, so lets replace our rails data with the json data, So firstly in our app.js.erb we'll replace `$scope.message = data` with `$scope.trail = data` to make it a bit more appropriate, and then back in our trails#show page we can replace eg

`<%= @trail.name %>`

with

`{{trail.name}}`

Now we have access to our users bookmark state from the received json, we can reflect that in the page, lets make a default state of notfaved and notcompleted, and then move our $http GET method into a function, so we can call it any time 


    $scope.favstate = "notfaved"
    $scope.compstate = "notcompleted"

    $scope.getTrailInfo = function () {
      $http.get($location.$$absUrl + '.json').success (function (data){
        $scope.trail = data
    })
    }
    
    $scope.getTrailInfo()
    
Now we can replace our ruby if statement, with angular, replacing

    <% if @bookmark.present? && @bookmark.favourited? %>
      <li><i class="fa fa-heart red"></i></li>
    <% else %>
      <li><i class="fa fa-heart-o"></i></li>
    <% end %>

with 

    <ul>                    
      <li ng-show="favstate == 'faved'"><i class="fa fa-heart red"></i></li>
      <li ng-show="favstate == 'notfaved'"><i class="fa fa-heart-o"></i></li>
      <li ng-show="compstate == 'completed'"><i class="fa fa-check circle green"></i></li>
      <li ng-show="compstate == 'notcompleted'"><i class="fa fa-check circle"></i></li>
    </ul>


(also showing the completed or not completed state of the trail bookmark). But at the moment our trails bookmark state is always 'notfaved' and 'notcompleted', so lets write a function to check the json for bookmark status and change the favstate/compstate accordingly

    $scope.checkUserBookmark = function (){
      if ($scope.userbookmark) {
        if ($scope.userbookmark.favourited == true) {
          $scope.favstate = "faved"
        }
        else{
          $scope.favstate = "notfaved"
        }
        if ($scope.userbookmark.completed == true) {
          $scope.compstate = "completed"
        }
        else{
          $scope.compstate = "notcompleted"
        }
      }
      else{
        $scope.favstate = "notfaved"
        $scope.compstate = "notcompleted"
      }
    } 
    
and now we can all that function from within our getTrailInfo function, 

      $scope.getTrailInfo = function (){
        $http.get($location.$$absUrl + '.json').success (function (data){
          $scope.trail = data
          $scope.checkUserBookmark(
        })
      }
      
So now the trail should once again show a red heart if faved and a white heart if not, but now we can write a function to change it when clicked, and to update the rails server with that info.

Lets go back to our html and turn our font-awesome white heart an angular click event, by changing 

`<i class="fa fa-heart-o"></i>` to `<i ng-click="addFav()" class="fa fa-heart-o"></i>`

so when the white heart is present (ie when trail is unfavourited by the user), it can be clicked and will run function addFav. Now lets write the addFav function. We have two scenarios, the bookmark model might not exist for this user, if its the first time the user has interacted with the trail, or it might exist, either because they had previously favourited it, but then unfavourited again, or because they had marked it as completed, and completed and favourited are both attributes of the same bookmark model. so we'll need to be able to either PUT to an existing bookmark, if it exists, or otherwise POST a new bookmark, if it doesnt.

If the bookmark doesnt already exist, we'll need to create everything from scratch, so we'll create a bookmark hash

      var letsBookmark = {}
      letsBookmark.trail_id = $scope.trail.id
      letsBookmark.user_id = $scope.trail.user.id
      letsBookmark.favourited = true
      
so our bookmark, will have the trail id of the trail, so we know which trail its bookmarking, and it will have the user id of the current user (careful here with naming, i have both trail.user_id as a direct attribute, for the trail uploader and trail.user.id, created by active model serializer, for the trail bookmarker)

and we can now POST our letsBookmark hash to /bookmarks, and if it succesfully posts, change our favstate to faved, which angular will then change immediately in the page to the red font-awesome heart, to show its been favourited 

    $http.post('/bookmarks.json', {bookmark: letsBookmark}).success (function (data){
      $scope.getTrailInfo()          
    })

now if the bookmark already exists, we already have trail_id, user_id and favourited status, we just need to change the favourited status from false to true

    $http.post('/bookmarks.json', {bookmark: letsBookmark}).success (function (data){
      $scope.getTrailInfo()         
    })
    
so our function should now look like this 

    $scope.addFav = function (){

      var letsBookmark = {}
      letsBookmark.trail_id = $scope.trail.id
      letsBookmark.user_id = $scope.trail.user.id
      letsBookmark.favourited = true

      if ($scope.trail.userbookmark) {         
        $http.put('/bookmarks/' + $scope.trail.userbookmark.id + '.json', {favourited: true}).success (function (data){
          $scope.getTrailInfo()            
        });
      }else{      
        $http.post('/bookmarks.json', {bookmark: letsBookmark}).success (function (data){
        $scope.getTrailInfo()
        })
      }
    } 
  

but when we click, we get the following error

    Started PUT "/bookmarks/4.json" for 127.0.0.1 at 2014-12-09 07:47:36 +0000
    Processing by BookmarksController#update as JSON
    Parameters: {"favourited"=>true, "id"=>"4", "bookmark"=>{"favourited"=>true}}
    WARNING: Can't verify CSRF token authenticity

so for now we'll add that into our rails bookmarks controller, above the index method

    skip_before_filter :verify_authenticity_token
    
Now we can favourite, but we cant unfavourite

so now lets go back to our html and add another click event, this time on the red heart, to unfavourite it. Now we could write a whole new function that would unfavourite it, or we could resuse the one we just wrote, and pass it an argument (whether its a faving action, or an unfaving action - 'favit' or 'unfavit')

    <li ng-show="favstate == 'faved'">
      <i ng-click="addFav('unfavit')" class="fa fa-heart red"></i>
    </li>
    <li ng-show="favstate == 'notfaved'">
      <i ng-click="addFav('favit')" class="fa fa-heart-o"></i>
    </li>

and now our addFav function should receive this argument

`$scope.addFav = function (f) `

and we could also use it for our completing and uncompleting, with a 3rd and 4th argument

     <ul>                    
       <li ng-show="favstate == 'faved'">
         <i ng-click="addFav('unfavit')" class="fa fa-heart red"></i>
       </li>
       <li ng-show="favstate == 'notfaved'">
         <i ng-click="addFav('favit')" class="fa fa-heart-o"></i>
       </li>
       <li ng-show="compstate == 'completed'">
         <i ng-click="addFav('undoneit')" class="fa fa-check circle green"></i>
       </li>
       <li ng-show="compstate == 'notcompleted'">
         <i ng-click="addFav('doneit')" class="fa fa-check circle"></i>
       </li>
      </ul>
      
and inside our addFav function, alter our attributes based on this 

    if (f == 'favit') {
      modit = {favourited: true}
      letsBookmark.favourited = true
    }else if (f == 'unfavit') {
      modit = {favourited: false}
    }else if (f == 'doneit') {
      modit = {completed: true}
      letsBookmark.completed = true
    }else if (f = 'undoneit') {
      modit = {completed: false}
    }


the letsBookmark it attributes are only going to be set to true (as as newly created bookmark doesnt have any true attributes to turn off), the modit hash can go in our PUT method, so

`$http.put('/bookmarks/' + $scope.trail.userbookmark.id + '.json', {favourited: true}).success (function (data)

can now become 

`$http.put('/bookmarks/' + $scope.trail.userbookmark.id + '.json', modit).success (function (data)
`
and now out favourite and completeing, unfavouriting and uncompleting work in real time, though we are making another trip to the rails server to do a GET request for the bookmark status, lets refactor that. But first lets also use angular to automatically update the total number of favs/completeds for the trail. Lets sign in as another user and bookmark a trail, then put `{{trail.bookmarks}}` somewhere on the page, and we should now see that the trail has 2 bookmark objects, users 1 and 4 have both favourited, and neither has completed

<img src="http://salterhebble.com/blogpics/ams4.jpg">

so now we can replace

`<%= @completeds.length %>  Hikers `

with 

`{{trail.bookmarks.length}}`

but this is now showing us the total number of bookmarks for the trail, regardless of favourited status, so lets filter that, and changed again to




and we can get rid of the if statements around this, so we should end up with

    <li><a href="#">{{(trail.bookmarks|filter: {completed:true}).length}}  Hikers <i class="fa fa-check circle green"></i></a></li>
    <li><a href="#">{{(trail.bookmarks|filter: {favourited:true}).length}} Hikers <i class="fa fa-heart red"></i></a></li>

#photos

lets finish off our trails#show page by converting the photos to being handled by angular too, but the photos model isnt yet part of our active model serializer, so lets add that in, which is just as simple as adding in the line `has_many :bookmarks` to the trail_serializer.rb

so now, we can change 

    <div class="mainphoto column-100"
      <%= image_tag(@trail.photos.first.image.trailpage, :class => "trailphoto column-100") %>>`

to

    <div class="mainphoto column-100" ng-repeat="photo in trail.photos|limitTo:1">
      <img class="trailphoto column-100" ng-src="{{photo.image.trailpage.url}}">
      
 

then we can also change the additional photos at the bottom of the page from

    <% @trail.photos.each do |photo| %>
      <div class="extraphoto column-25">
        <%= image_tag(photo.image.detail) %>
      </div>
    <% end %>
    
 to
 
    <div ng-repeat="photo in trail.photos" class="extraphoto column-25">
      <a href="{{photo.image.wide.url}}"><img ng-src="{{photo.image.detail.url}}"></a>
    </div>
    
 Next, we'll convert the index page and profile page to angular as well
 
 
 