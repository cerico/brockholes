---
layout: post
title: "$q and cordova file system   part II"
description: ""
category: 
tags: [cordoba, $q]
---
You remember in the last post we looked at introducing $q, now we can see how it really comes into its own, when we need to traverse the cordove file system, search for a file, upload it, and return the result. We want to keep all that logic out of the controller, and we actually want to keep most of it out of the Factory function that is exposed to the controller. Most of the logic is going to take place in internal functions within the factory. Lets look at our controller first, you'll remember from the last post, we were about to run a $scope.uploadAndSync function.

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


