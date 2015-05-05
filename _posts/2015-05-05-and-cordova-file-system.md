---
layout: post
title: "$q and cordova file system - part one"
description: ""
category: 
tags: [cordova,$q]
---

This is my first post since joining my new company, and I'm going to talk about $q and deferred promises, and about cordova file transfers. So, first, a little context. We have a sale object on the ipad, which has been posted to a Rails server, and it has an associated video, which needs to be updated at a later date. Our task here is to locate the video on the ipads filesystem and upload it to the rails server too.

{%highlight html%}
<h3>1. The View</h3>
{%endhighlight%}

The easiest part of the picture,

{% highlight html%}
<button ng-if="filtered.length > 0" class="button button-block button-balanced item item-icon-left" ng-click="syncAllSales()">Sync Sales</button>
 <ion-item ng-repeat="sale in sales">
  <span ng-show="!sale.videoSyncStatus"><i class="icon-1g assertive fa fa-video-camera"></i></span>
{% endhighlight%}

So here we have a syncing button which only shows if the list of filtered sales (thats is, sales needing some kind of attention) is greater than zero. Then we list  all sales, and if the video hasnt been uploaded we have have a font-awesome red camera next to it. Lets look at the syncAllSales function, which lives in our controller, mySalesCtrl.js


<h3>2. The controller, and the syncAllSales function</h3>


MySalesCtrl.js
{% highlight javascript%}
        $scope.syncAllSales = function(){
        for (var i = 0 ; i < $scope.sales.length; i++) {
            if (!$scope.sales[i].id) {
                if ($scope.sales[i].signature) {
                    SaleFactory.sendQuote($scope.sales[i]).then(function(data){
                        var sale = data.sale
                        $scope.uploadAndSync(sale)
                    })
                }
            }
            else if ($scope.sales[i].videoSyncStatus != true){
                var sale = $scope.sales[i];
                $scope.uploadAndSync(sale)
            }
        }
    };
{% endhighlight%}

This function is handling a number of things, firstly it is iterating through the sles currently stored on the ipad, to find out if they have an id or not (if they have an id, this means they have successfuly been posted to the server). In the first instance, we check if the sale has been signed for (ie - is this sale complete enough to post), if so we post the sale, and then we run the uploadAndSync function. In the second instance, we check if the sales video has been posted, and if not we also run the uploadAndSync function. But first lets come back to line 5, where we post the sale to the server, what is happening here?

<h3>3. The Factory, the sendQuote function and $q</h3>

We're currently in one particular controller, but we want as little logic in here as possible, as we may want to run the sendQuote function from elsewhere in our application. So we move it into a factory, 'SaleFactory', where it can be called from anywhere in the application, as long as we remember to inject it as a dependency

MySalesCtrl.js
{% highlight javascript%}
app.controller('MySalesCtrl', function ($scope, $state, SaleService,UploadService)
{% endhighlight%}

Now lets look at the sendQuote function in the SaleFactory factory

SaleFactory.js
                    {% highlight javascript%}
                    sendQuote: function(sale) {
                    var defer = $q.defer();
                    var server = $auth.apiUrl(); 
                    payload = {'amount': 0, 'token': ""};
                    var full_payload = {
                        "payload": JSON.stringify(payload),
                        "sale": JSON.stringify(sale)
                    };
                    createCharge(full_payload).then(function (result) {
                        defer.resolve(result)
                    });
                    return defer.promise
                },
                  {% endhighlight%}

Here we have a slightly cutdown version of the sendQuote function, but what is happening here? Well, firstly we can see that it isnt returning a value, its returning a promise, and secondly we can see that its running <i>another</i> function, createCharge. So whats going on?

If we remember back to our controller, we called the sendQuote function and added a .then on the end, like this

                    {% highlight javascript%}
                    SaleFactory.sendQuote($scope.sales[i]).then(function(data){
                        var sale = data.sale
                        $scope.uploadAndSync(sale)
                    })
                    {% endhighlight%}
                    
"then" means we are going to do something with the result "data", just as a .success does with a http.get, but in order to do this we need to return a promise not a value. I like to think of this like a plot in the ground in the garden, there's no plant there yet, but a space has been made. And we do this with $q. Our sendquote service can't return a value, because it doesnt what that value is yet, but it is going to return <i>some</i> value.

So lets go back to the sendQuote method - as we can see, it is running another function "createCharge" and its going to return the value of that, but it cant return that value before knowing what it is. createCharge is an internal function, it can only be called from within the SaleFactory itself, that is is to say - it isnt exposed to any controller (ie, it isnt with in the return {} block). We can think of these functions as being part of the internel logic of the factory, something the controller doesnt need to care about.

Lets look at the createCharge function, here is an edited version as it has a lot happening we don't need to care about right here, so for the purposes of this post, we are going to pretend the response.data.status is always 10

        {%highlight javascript%}
        function createCharge(full_payload){

          return $http.post($auth.apiUrl() + '/api/v1/sales/confirm_and_submit_payment', full_payload).success(function(response) {

                    if (response.data.status == 10)
                    {
                        sale = response.data.sale;
                        successfulSale(sale);
                        result = {status: 10, sale: sale}
                        return result
            }
          }
        }
        {%endhighlight%}
        
We can see that the function is posting the full_payload it received from the sendquote function, to the rails server and then it receieves a response, it then returns a result. But why don't we need $q here all of a sudden? and where is this result being returned <i>to</i>?

Firstly, we don't need $q here because we're using a $http method, which has $q-like features built in, it is returning a promise by default, its not something we have to explicitly build. And secondly, we're returning this to the sendQuote function that called it. When the createCharge returns its result, the sendQuote function picks it up here

{%highlight javascript%}
                    createCharge(full_payload).then(function (result) {
                        defer.resolve(result)
                       {%endhighlight%}
                       
and then fulfills its promise and returns it to the controller

{%highlight javascript%}
return defer.promise
{%endhighlight%}

and it is this which is returned to the controller

{%highlight javascript%}
               SaleService.sendQuote($scope.sales[i]).then(function(data){
                        sale = data.sale
                        $scope.uploadAndSync(sale)
                    })
{%endhighlight%}

Now in the controller, we can see that now the sale has been saved, it has an id and therefore the uploadAndSync function can now be run. We only want the uploadAndSync function to run when the sale has an id, and using promises allows us to call the uploadAndSync function in a .then for when it has resolved

In the next post, we can look at the uploadAndSync function and how promises can really simplify dealing with the cordova file system and transfer plugins                  
                    









