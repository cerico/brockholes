---
layout: post
title: "Ionic: Allowing users to display their own offline content in an iframe"
description: ""
category: 
tags: [rails, ionic, cordova]
summary: The app i'm working on at the moment has to have the ability to display 3rd party content in an iframe, and it must be able to do it offline. Our customers have to be able to upload a zipfile of a site to the rails server, and their ipad users must be able to download that zipfile, and be able to display it while offline, and they must be able to download and display multiple apps/products in the iframe.
---

Our task today is something of a complex multi-faceted one. The app i'm working on at the moment has to have the ability to display 3rd party content in an iframe, and it must be able to do it offline. Our customers have to be able to upload a zipfile of a site to the rails server, and their ipad users must be able to download that zipfile, and be able to display it while offline, and they must be able to download and display multiple apps/products in the iframe.

1 Rails

Lets start at the rails end, the customer has to be able to upload a zipfile of their product/application, the html, the css, the js, and any associated photos or videos. We also have to allow for the fact they may want to update existing products. I decided the best way to manage this was to allow a customer to has_many products, and to mark them as either active or inactive, and that only active products would be returned to the ipad via the API. I decided to derive the name of the product at the ionic end from the name of the zipfile. This presented a challenge, how to prevent the customer from mulitple uploads with the zipfile named the same.

In the end i decided to allow multiple products to have the same name, but only one active product to have that name, if you want to activate a product with a particular name that is already taken, the other product must be deactivated first. This has the additional benefit of keeping records of all active and discontinued products, withou any name changing. There is no 'update' as such, if a customer wants to update, they upload the new zipfile, give it the same name as its predecessor, then simply deactive its predecessor, ensuring only one active state of any given name

lets look at some code, first the view

companies/_form.html.erb

    {%highlight html%}
    <%= simple_form_for(@company) do |f| %>
    <%= f.error_notification %>

    <div>
      <%= f.label 'Add new package'%>
      <%= f.fields_for :products do |ff|%>
        <%= ff.file_field :file, class: 'form-control' %><br/>
      <% end %>
    </div>

    <div class="form-actions">
      <%= f.button :submit %>
    </div>
    <% end %>
    {%endhighlight%}

straightforward, just a regular upload form

companies_controller.rb

    {%highlight ruby%}
     def update
    @company.update(company_params)
    if @company.save
      if params[:company][:products] != nil
        @company.products << Product.create(file:(params[:company][:products][:file]),active:false,name:(params[:company][:products][:file].original_filename[/[^.]+/]))
      end
    end

    respond_with(@company)

    end
    {%endhighlight%}
    
Now lets look at making a product active

in the view, we have a simple checkbox, again straightforward

    {%highlight ruby%}
          <%= simple_form_for(@product) do |f| %>
          <%= f.error_notification %>

          <div class="form-inputs">
            <%= f.input :active, as: :boolean %>
          </div>


          <div class="form-actions">
            <%= f.button :submit %>
          </div>
      <% end %>
      {%endhighlight%}
      
and in the controller,

products_controller.rb

     {%highlight ruby%}
      def update
      @product = Product.find_by(id:params[:id])
      if params[:product][:active] == "0"
        @product.update_attributes(active:false)

        respond_with(@product)
      else
        if Product.where(name:@product.name,active:true,company_id:current_admin.company_id).length == 0
          if params[:product][:active] == "1"
          @product.update_attributes(active:true)
          respond_with(@product)
          end
        else
          flash[:error] = "Cannot have more than one product with same name active at any one time. Please deactivate older products first"
          respond_with(@product)
        end
      end
    end
    {%endhighlight%}
    
Some of this should move out into a helper method, but its staying here for the minute! If we're deactivating a product, its straightforward, we just turn it false, nothing else. But if we're activating a product we want to make sure its the only active product with that name, so we check there are no other active products with that name, belonging to the company of the current admin, and if there aren't any, we can turn this product active, and if there are we flash an error 

2 API 

Now lets look at the API

ap1/v1/products_controller.rb

this now makes the api controller's get_new_products method really simple! All it has to do is return all active products for that company, knowing there can't be any duplicates

    {%highlight ruby%}
      def get_new_products

    products = Product.where(company_id:current_user.company_id,active:true)

    render :json => products

    end
    {%endhighlight%}
    
Now lets move to the ipad, and have a look at the code there

3 ProductsController.js

        {%highlight javascript%}
        $scope.checkProducts = function(){
          ProductsService.downloadProducts().then(function(response){
              $scope.newPackages = response
          })
        }
        {%endhighlight%}

the controller calls the productservice's downloadproducts method, to display the new products available to download

4 ProductsService.downloadProducts

        {%highlight javascript%}
              downloadProducts:function(){

          var defer = $q.defer();
            checkServer().then(function(response){
              findCommon(response.data).then(function(alreadyHave){
                removeDiscontinued(alreadyHave,response.data).then(function(products){
                  localStorage.setItem("products", JSON.stringify(products));
                  insertNewProducts(alreadyHave,response.data).then(function(newProducts){
                    defer.resolve(newProducts)
                })
              })
            })
          });
          return defer.promise
        },
        {%endhighlight%}
        
quite a bit going on here with $q, which all happens inside the service, hidden from the controller then returns a fulfilled promise. So first it runs the checkserver function, feeds the response into the findCommon function, feeds the results of oth into the removeDiscontinued funtion, and then again into the insertNewProducts function, which returns the results we want to display, which we can then return to the controller, and therefore the view. Lets look at each in turn

5 checkServer

        {%highlight javascript%}
          function checkServer(){
          return $http.post($auth.apiUrl() + '/api/v1/products/get_new_products').success(function(response) {
        })
        }
        {%endhighlight%}
        
 self-explanatory, hits just hits our rails api to bring back an array of the currently active products, and this is fed into the findCommon function.
 
 6 findCommon
 
         {%highlight javascript%}
               function findCommon(activeProducts){
          var products = localStorage.getItem("products");//Retrieve the stored data
          products = JSON.parse(products); //Converts string to object
            var defer = $q.defer()
            var alreadyHave = [];
            for (var i=0; i < products.length; i++) {
              for (var j=0; j < activeProducts.length; j++) {
                if (products[i].id === activeProducts[j].id ) {
                  alreadyHave.push(products[i].id);
                }
              }
              defer.resolve(alreadyHave)
            }
        return defer.promise
        }
        {%endhighlight%}
        
This creates an empty array called 'alreadyHave', we iterated through both the current products and the active products array we received from the rails server, then push products commong to both into this new array, which we then return, to go into the removeDiscontinued and insertNewProducts functions

7 removeDiscontinued

        {%highlight javascript%}
              function removeDiscontinued(alreadyHave,activeProducts){

          var defer = $q.defer()
          for (var i = 0;i<products.length;i++){

              if (alreadyHave.indexOf(products[i].id) == -1){
                products[i].id == 5 ? products[i].active = true : products[i].active = false
                // above clears out inactive products, unless its product 5 - the super demo
                //todo - delete the html/css/js in cordova.data.directory?

              }else{
                }

            defer.resolve(products)
              }
          return defer.promise
          }
          {%endhighlight%}
          
  
 This iterates through our current products and if they are not in our new array, then this means they are discontinued, so they are set to active:false, and will no longer appear in our views. UNLESS its product with id 5, which is a dummy product we are keeping on the app for illustrative purposes
 
 8 insertNewProducts
 
         {%highlight javascript%}
         function insertNewProducts(alreadyHave,activeProducts){

          var defer = $q.defer()
          for (var i = 0;i <activeProducts.length;i++){
            if (alreadyHave.indexOf(activeProducts[i].id) == -1){
              activeProducts[i].need = true
            }else{
              activeProducts[i].need = false
            }
            defer.resolve(activeProducts)
          }
          return defer.promise
        }
        {%endhighlight%}
        
Finally we iterate through the array of active products from the server, and set them to need:true or false, depending if they appear in our common array, and therefore in our current product list

these are then returned back to our controllers checkProducts function, and hence the view, as 

      {%highlight javascript%}
      $scope.newPackages = response
      {%endhighlight%}
      
 9 the view
 
 if we now look in the view, we are returned a list of products we dont have and are currently active and therefor available to download
 
           {%highlight html%}
           <ion-item ng-repeat="package in newPackages | filter: {need:true}"
                    class="item-thumbnail-left" ng-click="getZip(package)">
                    {%endhighlight%}
                    
 we can filter the active products by a need:true, showing only the ones we dont already have, with an ng-click action on each, lets look at that 
 
 10 getZip
 
     {%highlight javascript%}
     $scope.getZip = function(package){
        package.syncing = true

      ProductsService.getZip(package.file.url).then(function(data) {
          ProductsService.addProduct(package).then(function (pack) {
              package.syncing = false
              package.downloaded = true
              package.active = true
              package.need = false
              $scope.products.push(package)
              ProductsService.getProducts().then(function(data){
                  $scope.products = data
              });
              //ProductsService.deleteZip()
          });

      });

    }
    {%endhighlight%}
    
    
 Here we have a function, which immediatley turns the syncing icon on, while a potentially large file is downloading, then we call the ProductsService.getZip function, then the ProductsService.addProduct function, and on completion ,the productsservice.getproducts function, lets look at prodctsservice.getZip
 
 11 ProductsService.getZip
 
         {%highlight javascript%}
                 getZip: function(package){
          var defer = $q.defer();
          downloadZip("",package).then(function(data){
            defer.resolve(data)
          });
          return defer.promise
        },
        {%endhighlight%}
        
 this calls the filesystem.downloadZip message, passing it an empty string, and the product name of what we would like to download, and passes the result to the controller (and hence the view), when done  
 
 12 downloadZip
 
         {%highlight javascript%}
               downloadZip: function (package) {
            var defer = $q.defer();
            var fileTransfer = new FileTransfer();
            var uri = encodeURI(server + package);
            var fileURL = cordova.file.dataDirectory + package;

            fileTransfer.download(
                uri,
                fileURL,
                function(entry) {
                  console.log("download complete: " + entry.toURL());
                  zip.unzip(fileURL,
                      cordova.file.dataDirectory + "products",
                      function(){
                        console.log('Zip decompressed successfully');
                        defer.resolve("done")
                      }
                  );
                },
                function(error) {
                  console.log("download error source " + error.source);
                  console.log("download error target " + error.target);
                  console.log("upload error code" + error.code);
                },
                false,
                {
                  headers: {
                    //"Authorization": "Basic dGVzdHVzZXJuYW1lOnRlc3RwYXNzd29yZA=="
                  }
                }
            );
            {%endhighlight%}
            
 Lots going on here, this is the cordova heart , where we set a download location, and then unzip to it, our zip files will come here, into the cordova file structure, each one into a directory with the same name as its zipfile, this means we know exactly where to link to from our view. Once this is complete, we run the PackageService.addProduct function
 
 13 addProduct
 
          {%highlight javascript%}
          addProduct: function(product){
          var defer = $q.defer()

          newproduct = {
          id: product.id,
              name: product.name.replace(/_/, " ").toLowerCase().capitalize(),
              packageUrl: cordova.file.dataDirectory + "products/" + product.name + "/index.html",
              img: 'img/jamesdemo.png',
              pricePence: 950,
            active:true
        },

          products.push(newproduct)
          localStorage.setItem("products", JSON.stringify(products));
          defer.resolve(products)
          return defer.promise
        },
        {%endhighlight%}
        
Here we create a new product, push it into the current products array, and then add it in localstorage, before returning the products array back to the controller

we set the packageUrl to the cordova.file.dataDirectory, plus the products subdirector we specifed all zips to go in, plus the product.name, and finally index.html, now back in the controller

        {%highlight javascript%}
              package.syncing = false
              package.downloaded = true
              package.active = true
              package.need = false
              {%endhighlight%}
              
 which means we can now set stasus's based on object state
 
     {%highlight html%}
               <ion-item ng-repeat="product in products  | filter:{active:true} "
                    class="item-thumbnail-left">
                    {%endhighlight%}
                    
 and now we display all the products with a filter, so we actually just show the active products
 
 

    
 
 








