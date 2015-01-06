---
layout: post
title: "Page Loading Spinner"
description: ""
category: 
tags: [angular,jquery]
summary: The trails with gpx data just take that little bit longer to load, with the extra data coming from the server. So here we'll add in a loading spinner
---
#spinner

I've put in a spinner gif, and put it in a div which via ng-show, will display when loading is true

1) app/assets/javascripts/templates/trailsindex.html
{% highlight html %}
<div class="spinner" ng-show="loading">
  <div class="bounce1"></div>
  <div class="bounce2"></div>
  <div class="bounce3"></div>
</div>
         
{% endhighlight%}

(the css for the bouncing ball is taken from [http://tobiasahlin.com/spinkit/](http://))

2) app/assets/javascripts/controllers/trailsindex.js.erb

{% highlight javascript %}
scenic.controller("TrailsIndexController", [ '$scope','$location','$http', function($scope,$location,$http) {
$scope.loading = true

$http.get('/trails.json').success (function (data) {
 $scope.trails = data
 $scope.loading = false
})
}])
{% endhighlight %}

This is reasonably effective, but will need refactoring to become false only when the photo is fully downloaded, so lazy loading is something to think about later. But we do have another problem which we might be able to use this for, the uploading of a new trail. The dropzone comes with its own progress loader for the image but that sticks at 100% once the image is done but the rest of the form is still processing, so lets have a look at that 

I had $scope.uploading = false on page load, and then $scope.uploading = true on submission of the form, but this didnt seem to work properly with the dropzone, so i revered to jquery to make this work. I got a different spinner from the site above, in the html simply used jquery as follows

{% highlight javascript%}
1  scenic.controller("NewController", [ '$scope','$location','uiGmapGoogleMapApi','Auth', function($scope,$location,uiGmapGoogleMapApi,Auth) {
2  $scope.loading = true;
3  $(".uploadingspinner").hide()

84  this.on("sendingmultiple", function() {
85  $(".uploadingspinner").show()
{%endhighlight%}

There's an angular progress bar module, which I'd like to use, so this part will probably get refactored at some point, but today, we stick with this


      









