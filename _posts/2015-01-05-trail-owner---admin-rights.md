---
layout: post
title: "Trail Owner   Admin Rights"
description: ""
category: 
tags: [rails]
summary: Each trail has an owner, and they should have admin rights over that trail, especially as any logged in user is able to upload photos to their trail, so in this post we'll give trail owners admin rights
---
First lets pass angular the current_user object

1) app/assets/javascripts/controllers/trail.js.erb

{% highlight javascript%}
$scope.current_user = current_user
{%endhighlight%}

and we'll use this to display the admin button, but first lets make it

2) css

{% highlight css %}
.sidebar li.admin{
background: rgb(255, 31, 31);
border: 2px solid white;
color: white;
border-radius: 3px;
padding: 9px;
text-transform: uppercase;
font-weight: 700;
}
{% endhighlight%}

3) app/assets/javascripts/templates/trail.html

{% highlight html%}
        <li class="admin" ng-click="administrateview('start')">Click to Admin your trail!</li>
        {% endhighlight%}
        
 and now we only want to show that if the current_user is the trail owner
 
4) app/assets/javascripts/templates/trail.html
 
 {% highlight html%}
          <li class="admin" ng-show="trailowner" ng-click="administrateview('start')">Click to Admin your trail!</li>
        {% endhighlight%}
        
5) app/assets/javascripts/controllers/trail.js.erb

{% highlight javascript %}
$scope.areYouTheOwner = function(){
  if ($scope.trail.user_id === current_user.id){
    $scope.trailowner = true
  }
}
{% endhighlight%}

so this function sets trailowner to true if the current user is the trailowner, so far so good. Lets also add in a state for when they have finished administering

7) app/assets/javascripts/templates/trail.html

{%highlight html%}
        <li class="admin" ng-show="trailowner" ng-click="administrateview('start')">Click to Admin</li>
        <li class="admin2" ng-show="administrating" ng-click="administrateview('finish')">Click to Finish</li>
        {% endhighlight%}


now lets write the administrateview function that the ng-click will run. We'll make this function turn on various admin functions on the page

6) app/assets/javascripts/controllers/trail.js.erb

{% highlight javascript %}

$scope.administrateview = function(admin){
  if (admin === "start"){
  $scope.deletePic = true
  $scope.deleteGPX = true
  $scope.makeFirstPic = true
  $scope.editName = true
  $scope.changeLoc = true
  $scope.administrating = true
  $scope.trailowner = false
}else if (admin === "finish"){
    $scope.deletePic = false
  $scope.deleteGPX = false
  $scope.makeFirstPic = false
  $scope.editName = false
  $scope.changeLoc = false
  $scope.administrating = false
   $scope.trailowner = true
}
}
{%endhighlight%}

Pretty self-explanatory, we're just changing states for various ng-show's we'll put into the template, so at the moment this will turn the two admin states on and off, and creating some new ns-show conditions, which we can now put in the html and write the corresponding functions. We'll cover 3 here, deletePic, makefirstPic and editName

#Deleting a photo

We'll use a fontawesome remove icon, and place it inside our ng-repeat, but outside the anchor tags

7) app/assets/javascripts/templates/trail.html

{% highlight html%}

    <div ng-repeat="photo in trail.photos" class="extraphoto column-25">
      <i ng-show="deletePic" ng-click="deletePhoto(photo)" class="fa fa-times fa-2x"></i>
        <a class="fancybox" rel="group" href="{{photo.image.wide.url}}"><img ng-src="{{photo.image.detail.url}}"></a>
    </div>
             {% endhighlight%}
             
8) {% highlight javascript%}

$scope.deletePhoto = function (photo){
  console.log(photo)
}
{% endhighlight%}

if this shows us the photo correctly (it does), then we can send this to be deleted.

9) {% highlight javascript %}

$scope.deletePhoto = function (photo){
  $http.delete('/photos/' + photo.id + '.json').success (function (){
    console.log("its gone")
  })
}
{% endhighlight%}

But we want to protect this at the server end, because "never trust the client". So once this is received by the rails server, we need to make sure we only delete the photo if a) the user is authorized, and b) its not the only photo the trail has (as we want each trail to have at least one photo)

10) app/controllers/photos_controller.rb

{% highlight ruby%}
def destroy

  @photo = Photo.find(params[:id])
    if current_user.id === Trail.find_by_id(@photo.trail_id).user_id && Trail.find_by_id(@photo.trail_id).photos.length > 1
    @photo.destroy

    respond_to do |format|
        format.html { redirect_to photos_url }
        format.json { head :no_content }
      end
    end
  end
{% endhighlight%}

And now we want the photo to disappear from the current view as well, so first we need to find out where in the array the photo is

11) app/assets/javascripts/controllers/trail.js.erb

{% highlight javascript%}
$scope.deletePhoto = function (photo){
  $http.delete('/photos/' + photo.id + '.json').success (function (){
    console.log($scope.trail.photos.indexOf(photo))
  })
}
{% endhighlight%}

and once we have that we can splice the photo from the array, and it should disappear from the page

12) app/assets/javascripts/controllers/trail.js.erb

{% highlight javascript%}
$scope.deletePhoto = function (photo){
  $http.delete('/photos/' + photo.id + '.json').success (function (){
    var index = $scope.trail.photos.indexOf(photo)
    $scope.trail.photos.splice( index, 1 );
  })
}
{% endhighlight%}

If we delete the first photo in the array, the main photo will automatically be replaced by the new first in the array, but lets also write a function where the trail owner can change the main photo without deleting anything

#Making a photo the main photo

13) app/assets/javascripts/templates/trail.html

{% highlight html%}
  <span class="makemain" ng-show="makeFirstPic" ng-click="makeMain(photo)">Make Main Photo</span>
  {%endhighlight%}
  
14) css

{% highlight css%}
.makemain{
float: left;
font-size: 16px;
background: rgb(85, 85, 181);
color: white;
padding: 7px;
font-family: 'raleway';
margin-top: -1px;
margin-bottom: 2px;
text-transform: uppercase;
}
{%endhighlight%}


15) app/assets/javascripts/controllers/trail.js.erb

{% highlight javascript%}
$scope.makeMain = function (photo){
  $http.put('/photos/' + photo.id + '.json', {trailid:photo.trail_id}).success (function(){
    console.log(photo)
  })

}
{% endhighlight%}

So here, im sending the photo's id to the rails server, now lets look at the rails end

16) binding.pry in app/controllers/trails_controller.rb#update

{%highlight ruby%}
[1] pry(#<TrailsController>)> params
=> {"trailid"=>151, "action"=>"update", "controller"=>"trails", "id"=>"36", "format"=>"json", "trail"=>{}}
{%endhighlight%}

Originally I was planning to change the order either at the rails end, or back in the javascript, but given that the photos have a, till now, unused name attr_accesible, i decided to use that instead. I'm not particularly happy with this section, so this will be an area for refactoring later, but for now

17) app/assets/javascripts/templates/trail.html

{% highlight html%}
      <div class="mainphoto column-100" ng-repeat="photo in trail.photos|filter: {name:'main'}" 
      {% endhighlight%}
      
changed this from limitTo:1, to filtering by name "main"


18) app/assets/javascripts/controllers/trail.js.erb

{%highlight javascript%}
$scope.makeMain = function (photo){
  $http.put('/photos/' + photo.id + '.json', {trailid:photo.trail_id}).success (function(){
    console.log(photo)
    for (var i = 0;i < $scope.trail.photos.length; i++){
      $scope.trail.photos[i].name = "other"
    }
    photo.name = "main"
})

}
{% endhighlight%}

I'm just setting the name of all photos belonging to the trail to other, and then the photo in question gets named main. Of course we'll need to now handle that at the server side too

19) app/controllers/photos_controller.rb

{%highlight ruby%}
    def update
    if current_user.id === Trail.find_by_id(@photo.trail_id).user_id 
      Photo.where(trail_id:params[:trailid]).each do |photo|
        photo.name = "other"
        photo.save
      end  
      @photo = Photo.find(params[:id])
      @photo.name = "main"
    end
   
     {% endhighlight%}

this also meant a refactoring for the delete code, as now automatic replacement of first photo in the array will no longer occur, so the easiest thing is just to prevent deletion of trail.photo with name of "main", to delete that photo a trailowner will have to make another photo the main photo and then deleted the previous main photo, so lets change that

20) app/controllers/photos_controller.rb

{% highlight ruby %}

  def destroy

    @photo = Photo.find(params[:id])
    if current_user.id === Trail.find_by_id(@photo.trail_id).user_id && @photo.name != "main"
      @photo.destroy
      {% endhighlight%}
      
so here, im just preventing deletion of any photo with a name of "main", and same thing in the angular, we'll pop up a warning that trailowner wont be able to delete main photo

21) app/assets/javascripts/controllers/trail.js.erb

{% highlight javascript %}
$scope.deletePhoto = function (photo){
  $http.delete('/photos/' + photo.id + '.json').success (function (){
    var index = $scope.trail.photos.indexOf(photo)
    $scope.trail.photos.splice( index, 1 );
  }).error (function(){
    alert("Can't delete main trail photo!")
  })
}
{% endhighlight %}

#return to active model serializer

this has had an effect on the index page, so i thought there were two ways to solve this, either refactor for the index page to show trail.photos|filter: {name:'main'} instead of trail.photos[0], or make use of the active model serializer to return a mainphoto json, so i decided on the second (we'll also do the same in the custom serializer)

22) app/serializers/trail_serializer.rb and app/serializers/points_serializer.rb

{% highlight ruby %}
1  class TrailSerializer < ActiveModel::Serializer
2  attributes :id, :name, :county, :rating, :user_id, :distance, :lat, :lng, :hikerbookmark, :hiker, :mainphoto

15  def mainphoto
16    photos.find_by_name("main")
17  end
{% endhighlight%}

so im just adding mainphoto to the list of attributes on line 2, and then all mainphoto does is return whichever photo for each trail has a name of "main"

23) app/assets/javascripts/templates/trailsindex.html

<img src="http://salterhebble.com/blogpics/trailowner1.jpg">
          
 
 and refactoring here, to use the mainphoto json instead of photos[0]
 
24) app/assets/javascripts/templates/trail.html

<img src="http://salterhebble.com/blogpics/trailowner2.jpg">

and here we can replace the ng-repeat and just use the mainphoto json

we also need to refactor the makeMain function, which can now become a little simpler

25) app/assets/javascripts/controllers/trail.js.erb

{%highlight javascript%}
$scope.makeMain = function (photo){
  $http.put('/photos/' + photo.id + '.json', {trailid:photo.trail_id}).success (function(){
    $scope.trail.mainphoto = photo
  })
}
{% endhighlight %}

#change hike name

First lets make a form which appears when in admin mode

26) app/assets/javascripts/templates/trail.html

{%highlight html%}
    <div class="periedit" ng-show="administrating">
       <form class="trailedit" ng-submit="editname()">
      <input type="text" placeholder="Trail Name" ng-model="newtrailname"  size="30">
      <input class="edit" type="submit" value="add">
      </form>

    </div>
  {%endhighlight%}
  
27) css

{%highlight css%}
.periedit {
  background: rgba(0,0,0,0.74);
line-height: 100%;
}

.trailedit {
  display: inline;
margin-left: 8px;
}


.edit {
background: rgb(88, 140, 176);
width: 121px;
margin-left: 4px;
color: white;
line-height: 103%;
font-size: 19px;
padding: 3px;
padding-left: 4px;
text-transform: uppercase;
border: 2px solid white;
border-radius: 5px;
}
{%endhighlight%}

28) app/assets/javascripts/controllers/trail.js.erb

{% highlight javascript %}
$scope.editname  = function(){
  console.log($scope.newtrailname)
  $scope.trail.name = $scope.newtrailname
}
{% endhighlight%}

which should change the trail name when submitted, but of course we need to send that to the server so that its changed for real, and only then change it in the view

29) app/assets/javascripts/controllers/trail.js.erb
{% highlight javascript %}
$scope.editname  = function(){
  console.log($scope.newtrailname)
    var newtrailname = {}
  newtrailname.name = $scope.newtrailname


  $http.put($location.path() + '.json', newtrailname).success (function(){
  $scope.trail.name = $scope.newtrailname
})
}
{% endhighlight%}

so here i'm passing the newtrailname as part of a hash, but at the rails trail#update end we need to modify, because currently the only updates to a trail involve photos, so I'm putting in an if statement, depending on whether the update receives either a photo in the params, or a newtrailname

30) app/controllers/trails_controller.rb (#update)

{%highlight ruby%}
  def update

    @trail = Trail.find(params[:id])
    if current_user.id === @trail.user_id 
      if params[:photo]     
        Photo.where(trail_id:params[:id]).each do | photo|
          photo.name = "other"
        end
        @photo = Photo.find_by_id(params[:photo])
        @photo.name = "main"
      else

        respond_to do |format|

          if @trail.update_attributes(params[:trail])

        # format.html { redirect_to @trail, notice: 'Trail was successfully updated.' }
        format.json { render json: @trail, serializer: PointsSerializer, root: false  }
      else
        format.html { render action: "edit" }
        format.json { render json: @trail.errors, status: :unprocessable_entity }
      end
    end
  end
end
end
{% endhighlight%}

#ngScrollTo

The admin buttons for the deleting a photo or making it the main photo are towards the bottom of the page and not immediately obvious like the trail rename button, so lets make all our admin buttons at the top of the page, and use them to navigate to the part of the page we want, lets start by injecting $anchorScroll into the trail controller.

31) app/asset/javascripts/controllers/trail.js.erb

{% highlight javascript %}
scenic.controller("TrailsController", ['$scope','$location','$http','uiGmapGoogleMapApi','Auth','$anchorScroll', function($scope,$location,$http,uiGmapGoogleMapApi,Auth,$anchorScroll) {

   $scope.scrollTo = function(id) {
    $location.hash(id);
    $anchorScroll();
  }
{% endhighlight%}

32) app/assets/javascripts/templates/trail.html

{% highlight html%}
        <div class="editphotos" <a href="" ng-click="scrollTo('editphotos')">edit photos</a></div>
        <div class="editlocation"><a href="" ng-click="scrollTo('editmap')">edit location</a></div>
        
        
           <div id="editphotos"></div>
              <div id="editmap"></div>
{%endhighlight%}

just place the divs in the correct location on page, so now we just need to write the edit location function, so a trailowner can change the mainmarker of the map of their trail

#edit location and edit gpx

But as this is a bit of a longer post, we'll just leave placemarkers for that, and do that in the next post

31) app/assets/javascripts/templates/trail.html

{%highlight html%}
  <div class="editgpx" ng-show="ifgpx" ng-click="deletegpx()">delete gpx</div>
        
  <div id="editmap"><span class="editingmap" ng-show="makeFirstPic" ng-click="editmap()">Edit Location</span>
                </div>
        {% endhighlight%}
        
32) app/assets/javascripts/controllers/trail.js.erb

{%highlight javascript%}
      if ($scope.trail.points.length > 1){
        $scope.ifgpx = true
      }
      
 $scope.deletegpx = function(){
  alert("coming soon")
}

$scope.editmap = function(){
  alert("coming soon!")
}
{% endhighlight%}



