---
layout: post
title: " and cordova file system"
description: ""
category: 
tags: [cordova,$q]
---

This is my first post since joining my new company, and I'm going to talk about $q and deferred promises, and about cordova file transfers. So, first, a little context. We have a sale object on the ipad, which has been posted to a Rails server, and it has an associated video, which needs to be updated at a later date. Our task here is to locate the video on the ipads filesystem and upload it to the rails server too.

{%highlight html%}
<h3>The View</h3>
{%endhighlight%}

The easiest part of the picture,

{% highlight html%}
<button ng-if="filtered.length > 0" class="button button-block button-balanced item item-icon-left" ng-click="syncAllSales()">Sync Sales</button>
 <ion-item ng-repeat="sale in sales">
  <span ng-show="!sale.videoSyncStatus"><i class="icon-1g assertive fa fa-video-camera"></i></span>
{% endhighlight%}
