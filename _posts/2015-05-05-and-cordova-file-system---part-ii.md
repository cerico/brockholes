---
layout: post
title: "$q and cordova file system   part II"
description: "You remember in the last post we looked at introducing $q, now we can see how it really comes into its own, when we need to traverse the cordova file system, search for a file, upload it, and return the result."
category: 
summary: You remember in the last post we looked at introducing $q, now we can see how it really comes into its own, when we need to traverse the cordova file system, search for a file, upload it, and return the result.
tags: [cordoba, $q]
---
You remember in the last post we looked at introducing $q, now we can see how it really comes into its own, when we need to traverse the cordova file system, search for a file, upload it, and return the result. We want to keep all that logic out of the controller, and we actually want to keep most of it out of the Factory function that is exposed to the controller. Most of the logic is going to take place in internal functions within the factory. Lets look at our controller first, you'll remember from the last post, we were about to run a $scope.uploadAndSync function.

    {%highlight javascript%}
    $scope.uploadAndSync = function(sale){
            sale.syncing = true;
        UploadFactory.sendCurrentFile(sale).then(function(data){
            sale.syncing = false;
            SaleService.synchedVideo(sale)
            $scope.sales = SaleService.getAllSales();
        });
    }
    {%endhighlight%}
    
Our function is passed a sale object, and we turn sale.syncing true (this is only to show a spinning wheel in the view, nothing more). Then we run the sendCurrentFile function, which is in the UploadFactory. There's that .then again, we're only going to run the rest of the functions once sendCurrentFile returns a result, or as you probably guessed, when it fulfills its promise.

Lets look at the sendCurrentFile function, which we can see is the only function in the factory that is returned/exposed to the controller

        {%highlight javascript%}

        return {
            sendCurrentFile: function(sale){
                var defer = $q.defer();
                 window.fileStorage.getFileSystem(sale).then(function(file){
                        window.fileStorage.uploadVideo(file,sale).then(function(result){
                            defer.resolve(result);
                        })
                    });
                return defer.promise
            }
          }
          {%endhighlight%}
          
We receive a sale object from the controller, which we then pass to an internal function, which weve called nonExposed.getFileSystem, the result of which we then pass to the nonExposed.uploadVideo function - and its the result of <i>that</i> which we pass back to the controller. Put more simply, the first function searches cordova's filesystem for a particular file, and <i>then</i> the file it finds is uploaded via the uploadVideo function, then we can return the result of all that to the controller

Lets first look at the getFileSystem function 

{%highlight javascript%}
            window.fileStorage = {
              getFileSystem: function(sale){
                var defer = $q.defer();
                var fileSystem = function(fileSystem){
                    var directoryReader = fileSystem.root.createReader();
                    directoryReader.readEntries(findAFile);
                };
                var findAFile = function(entries) {
                    var thisfile = sale.filename + ".mp4";
                    for (var i = 0; i < entries.length; i++) {
                        if (entries[i].name == thisfile) {
                            defer.resolve(entries[i])
                        }
                    }
                };
                var fail = function(error){
                    console.log(error)
                };
                window.requestFileSystem(LocalFileSystem.PERSISTENT, 0, fileSystem, fail);
                return defer.promise
            },
          }
{%endhighlight%}

This function just returns the file to be uploaded, again we have the defer.resolve and the defer.promise. First we request the filesystem, and that is passed a required ileSystem fuction, which creates a directory reader - and <i>that</i> is passed the findAFile function, which iterates through everything it finds in the root directory, and searches for an mp4 with the sale object's filename value). If it finds a file with this filename, it resolves it and the promise is fulfilled and return back to the sendCurrentFile function. Lets go back and look at that

    {%highlight javascript%}

        return {
            sendCurrentFile: function(sale){
                var defer = $q.defer();
                 window.fileStorage.getFileSystem(sale).then(function(file){
                        window.fileStorage.uploadVideo(file,sale).then(function(result){
                            defer.resolve(result);
                        })
                    });
                return defer.promise
            }
          }
          {%endhighlight%}
 
 So now we have had the file object passed back to us, we can now run the uploadVideo function, passing it the sale we were originally passed, and now also the file we just received, so again we go back into the internal or non-exposed part of the factory and run this function, lets look at it (again, truncated to just the relevant parts)
 
 {%highlight javascript%}
 
             uploadVideo:function(file,sale){

                var defer = $q.defer();
                var ft = new FileTransfer()
                var path = file.nativeURL
                var name = file.name;
                var server = $auth.apiUrl() + "/api/v1/sales/update_with_video";
                var uploadSuccess = function(result) {
                        defer.resolve(file);
                    file.remove(function(file){
                        console.log("file successfully removed");
                    });
                };
                ft.upload(path, server, uploadSuccess, uploadError, {});
                };
                return defer.promise
            },
            {%endhighlight%}
 
 So again we can see the defer.resolve and defer.promise in play, it will only return the result, once a result has been arrived at, so we pass it the file we were returned in our previous function, and this is uploaded to our rails API endpoint. What we are interested in here is the uploadSuccess function, once this file has transferred to the rails server successfully, it is resolved (and removed, in our case!) - and the promise is fulfilled and the result returned back to the sendCurrentFile function.

and now if we look back at that again, 

    {%highlight javascript%}

        return {
            sendCurrentFile: function(sale){
                var defer = $q.defer();
                 window.fileStorage.getFileSystem(sale).then(function(file){
                        window.fileStorage.uploadVideo(file,sale).then(function(result){
                            defer.resolve(result);
                        })
                    });
                return defer.promise
            }
          }
          {%endhighlight%}
          
we can see that it is only <i>now</i> that the result can be resolved, and this functions <i>own</i> promise can can be fulfilled, and finally returned back to the controller

And now looking back to our controller


     {%highlight javascript%}
    $scope.uploadAndSync = function(sale){
            sale.syncing = true;
        UploadFactory.sendCurrentFile(sale).then(function(data){
            sale.syncing = false;
            SaleService.synchedVideo(sale)
            $scope.sales = SaleService.getAllSales();
        });
    }
    {%endhighlight%}
    
 now that the promise has been resolved, the syncing process is over and sycning can be set to false, and we can run any other functions we want, knowing that the video has succesfully uploaded, here we have some other functions that should only be run against a sale, once its video has been uploaded and saved
 
<h3>Rails API</h3>

Lets briefly have a little look at what happens at the server end, when the file arrives

{%highlight ruby%}
 
   def update_with_video
    @sale = Sale.find_by_id(params[:file].original_filename)
    @sale.update(video:params[:file],hasvideo:true)
    result = {status: true, message: 'Success',sale: @sale}
    render :json => result
  end
  
  {%endhighlight%}
  
Here we an see the endpoint, we can find the sale because the filename of the video has that as its id, so we can now update the Sale's video parameter with the file (via carrierwave), and set the hasvideo boolean to true, we can then return the result back to our Ionic app as json 

<h3>Recap</h3>

Lets take a step back, and see how this works in an overview, we'll take out anything to do specificaly with Cordova and put some arbitrary functions in

{%highlight javascript%}
app.factory('calderFactory', ['$http','$q', function ($http,$q) {

        function firstfunction(rain){
            var defer = $q.defer();
             COMPLICATEDSTUFF.then(result){
                defer.resolve.(result)
            }
            return defer.promise
        }
        function secondfunction(rain,wind){
            var defer = $q.defer();
            COMPLICATEDSTUFF.then(result){
                defer.resolve.(result)
            }
            return defer.promise
        }
    function thirdfunction(rain,wind){
        var defer = $q.defer();
        COMPLICATEDSTUFF.then(result){
            defer.resolve.(result)
        }
        return defer.promise
    }
        

        return {
            getWeather: function (rain) {
                var defer = $q.defer();
                firstfunction(rain).then(function (wind) {
                    secondfunction(rain, wind).then(function (traffic) {
                        thirdfunction(rain, wind, traffic).then(function (happiness) {
                            defer.resolve(happiness);
                        })
                    })
                });
                return defer.promise
            }
        }
    }]);

    app.controller('MyController', function ($scope, $state, CalderFactory,RibbleFactory) {

        $scope.basicFunction = function (rain) {
            weather.checking = true;
            calderFactory.getWeather(sale).then(function (data) {
                weather.checking = false;
                ribbleFactory.recordResult(data)
            });
        }
    };

{%endhighlight%}

what we have here are 3 internal factory functions which do 'COMPLICATEDSTUFF', and are called in turn by the getWeather function in the factory, and this is the only function exposed to the controller. We can think of the returned 'getWeather' function in the factory as controller-facing, or customer-facing, like a waiter, while the factory's other functions can be thought of as like chefs, the controller cant call them directly, the returned getWeather function is merely presenting the work done by the chef functions


