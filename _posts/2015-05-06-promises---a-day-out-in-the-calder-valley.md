---
layout: post
title: "Promises - A day out in the calder valley"
description: ""
category: 
summary: The premise of this little app, is we want to go on a day out visiting some towns that serve Little Valley Ale. Ideally we'd like to stay in yorkshire, so we'll check there, and if we don't have enough towns, only then will we look further afield, in Lancashire
tags: [$q, promises]
---

Lets have another look at promises, and we can see a real example here, at [http:/calder.io/promises](http://calder.io/promises)

The premise of this little app, is we want to go on a day out visiting some towns that serve Little Valley Ale. Ideally we'd like to stay in yorkshire, so we'll check there, and if we don't have enough towns, only then will we look further afield, in Lancashire

1. The view

{%highlight html%}

          number of towns we'd like to visit = {{required_towns}}
               <p class="button green towns back-link-honley" ng-click="visit(2)">2</p>
           <p class="button green towns back-link-honley" ng-click="visit(3)">3</p>
           <p class="button green towns back-link-honley" ng-click="visit(4)">4</p>
          yorkshire towns:         
            <ul><li ng-repeat="town in yorkshire_towns">{{town.name}}</li></ul> 
               
          lancashire towns: 
             <ul><li ng-repeat="town in lancashire_towns">{{town.name}}</li></ul> 

            do we have enough towns? = {{enough_towns}}<br>
               <ul>
                towns serving Little Valley Ales: 
            <li ng-repeat="town in towns_serving_little_valley">{{town.name}}</li></ul> 
{%endhighlight%}

all fairly self-explanatory here, we're going to show the number of towns the user wants to visit, the numbe of towns in yorkshire, the number of towns in lancashire and the number of towns serving our favoured ale, the user can click on a button to choose how many towns

2. The controller - part i

{%highlight javascript%}

app.controller('DayOutCtrl', function ($scope, $state,littleValleyService,LancashireService,YorkshireService,$q) {

{%endhighlight%}

here we are just injecting our factories (and also $q!), now we have 3 main functions, a yorkshire pub finder, a lancashire pub finder, and a function to state just how many towns we want to drink in. Lets look at each in turn

3. the controller - part ii (getLocalPubs)

{%highlight javascript%}
  $scope.getlocalPubs = function(){
    var defer = $q.defer()
    YorkshireService.getyorkshireTowns().then(function(data){
      $scope.yorkshire_towns = data
      for (var i =0 ; i <$scope.yorkshire_towns.length ;i++ ){
        town = $scope.yorkshire_towns[i]
        littleValleyService.serveLittleValley(town).then(function(data){
          $scope.towns_serving_little_valley.push(data)
        })
      }
      defer.resolve("finished")
    })
    return defer.promise
  }

{%endhighlight%}

this function, will return a promise when it is called, because it doesnt know the result itself, it has to call an external service to get the data, and it can return a result, once it gets something back from the controller. Once it has received the yorkshire towns from the yorkshire service it can pass each town to the littleValleyService to see if the ale they serve is a Little Valley ale, but this functions isn't called directly, its called by the visit function, where the tourist tells us how many towns they want to visit, so lets look at that 

4. the controller - part iii (visits)

{%highlight javascript%}
 $scope.visit = function(number){
    $scope.enough_towns = "looking locally"
    $scope.towns_serving_little_valley = []
    $scope.required_towns = number
    $scope.getlocalPubs().then(function(){
      if ($scope.towns_serving_little_valley.length < $scope.required_towns){
        $scope.enough_towns = "not enough places serve our beer - we're going to have to look in lancashire too"
        $scope.getregionalPubs()
      }else{
        $scope.enough_towns = "we can do this day out, all in yorkshire!"
      }
    })
  }
  {%endhighlight%}

So,  here we found out how many towns the tourist wants to visit, and then we run the getLocalPubs function, as seen above - and as that returns a promise, we are able to add a .then to the end, and do something with the result <i>when</i> its fulfilled, so in this case if the getLocalPubs doesnt return the requisite number of towns, then it will run the getregionalPubs function, but it will only decide whether to run this function once the getlocalPubs function has fulfilled its promise, and not before

5 the controller - part iii (getRegionalPubs)

{%highlight javascript%}
  $scope.getRegionalPubs= function (){
   LancashireService.getLancashireTowns().then(function(towns){
    $scope.lancashire_towns = towns
    for (var i =0 ; i <$scope.lancashire_towns.length ;i++ ){
      town = $scope.lancashire_towns[i]
      littleValleyService.serveLittleValley(town).then(function(lancstown){
        $scope.towns_serving_little_valley.push(lancstown)
        if ($scope.towns_serving_little_valley.length > $scope.required_towns){
          $scope.enough_towns = "we can do this but we'll have to go into lancashire too"
        }else{
          $scope.enough_towns = "still not enough towns, lets call it off"
        }
      })
    }

  })
 }

{%endhighlight%}

now this function is only run if getLocalPubs doesnt return enough towns serving a Little Valley ale. It functions much the same way as the getLocalPubs function, it gets some lancashire towns from the lancashire factory, and feeds them to the littleValleyService factory to see if they serve the right kind of ale, and then it adds towns that meet the requirement to the towns_serving_little_valley array, and then we can see if we can do this trip by going a little further afield. Again you can see the use of the .then on each factory, as they are all returning promises too, so lets look at those next

6. Factories i - Yorkshire

Here we have the yorkshire factory, which simply returns a list of yorkshire towns, and the one ale that they serve, wrapped in a timeout to simulate a trip to a server, returning a promise which is fulfilled when the timeout finishes

    {%highlight javascript%}
    angular.module('pendle.services')
    app.factory('YorkshireService', ['$http','$q','$timeout', function ($http,$q,$timeout) {

    var yorkshire_towns = [{"name":"slaithwaite","ale":"withens"},
                           {"name":"marsden","ale":"thwaites"},
                           {"name":"brockholes","ale":"clough"},
                           {"name":"todmorden","ale":"slack"},
                           {"name":"denby dale","ale":"denby pale ale"},
                           {"name":"mytholmroyd","ale":"walshaw"}
                           ]
    
      return {
       getyorkshireTowns: function(){
        var defer = $q.defer()
        $timeout(function(){
          defer.resolve(yorkshire_towns)
        },3000)       
        return defer.promise
       }
      }

  
     }]);
     {%endhighlight%}
     
7. Factories ii - Lancashire

and the lancashire factory does the same thing,

    {%highlight javascript%}
    angular.module('pendle.services')
    app.factory('LancashireService', ['$http','$q','$timeout', function ($http,$q,$timeout) {

      var lancashire_towns = [{"name":"bacup","ale":"ormerod"},
                             {"name":"colne","ale":"thwaites"},
                             {"name":"darwen","ale":"bleaksmoor"},
                             {"name":"accrington","ale":"cragg"},
                             {"name":"rawtenstall","ale":"miners broth"},
                             {"name":"stacksteads","ale":"withens"}
                             ]

    return{
       getLancashireTowns: function(){
         var defer = $q.defer()
         $timeout(function(){
         defer.resolve(lancashire_towns)
       },6500)
         return defer.promise
      }


    }
    }]);

    {%endhighlight%}
    
8. Factories iii - Little Valley

the controller's feed the results they get back from the yorkshire and lancashire services, into the littlevalley service, to see if the ale that is served in each town is a Little Valley ale. lets look at that

    {%highlight javascript%}
    angular.module('pendle.services')
    app.factory('littleValleyService', ['$http','$q', function ($http,$q) {

      littlevalleyales = ["withens","slack","craggvale"]


      return {
        serveLittleValley: function(town){
        var defer = $q.defer()
        if(littlevalleyales.indexOf(town.ale) !== -1){
            defer.resolve(town)
         } 
        return defer.promise 
       }
      }
      

  
    }]);

    {%endhighlight%}
 
Here we have 3 different types of Little Valley ales, and the serveLittleValley function checks each town it receives from the controller, to see if its ale is a little valley ale (using indexOf to check its position in the array), once done it resolves its promise, and the results are returned to the controller
 
Here we can see why promises are used, we can only evaluate certain things once events have run their course. We don't want to check the lancashire towns, until the yorkshire towns have been checked, and then we can save ourselves an unneccessary trip to the server, that the lancashire service is simulating

Mine's a Withens Top please - don't forget to go have a look at this in practive at <a href="http://calder.io/promises">http://calder.io/promises</a>





