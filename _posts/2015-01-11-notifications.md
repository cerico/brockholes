---
layout: post
title: "notifications"
description: ""
category: 
tags: [notifications, angularjs]
---

Lets have the app notify a trailowner when a modification has been made to their trail. Any logged in user is able to upload photos to a trail, so when that occurs lets have the trail owner notified. To do this we can compare current\_user sign in time, to @trail.update\_at time. Currently we're not actually updating the trail model when a new photo is created, so firstly we'll need to sort that out

1) app/controllers/photo_controller.rb

{%highlight ruby%}
          if @photo.save
         
             @trail.update_attributes(lastphoto:@photo.id)

{%end}

Here i'm updating the lastphoto param with the id of the just uploaded photo, this will also ensure the updated_at param will be correct. Now we can run the following in our controller to show notifications for trails that have had a photo added since the trail owner last logged in

2) app/assets/javascripts/

{%highlight javascript%}
$scope.notifications = 0
  for (var i = 0; i<$scope.user.trails;i++){
    if ($scope.user.trails[i].updated_at < current_user.updated_at) {
      $scope.notifications += 1
    }
  }
{%endhighlight%}

