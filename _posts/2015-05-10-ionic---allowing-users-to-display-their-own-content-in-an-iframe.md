---
layout: post
title: "Ionic: Allowing users to display their own content in an iframe - part i: rails"
description: ""
category: 
tags: [rails, ionic, cordova]
---

Our task today is something of a complex multi-faceted one. The app i'm working on at the moment has to have the ability to display 3rd party content in an iframe, and it must be able to do it offline. Our customers have to be able to upload a zipfile of a site to the rails server, and their ipad users must be able to download that zipfile, and be able to display it while offline, and they must be able to download and display multiple apps/products in the iframe.

1. Rails

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

2. API 

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

3. ProductsController.js

        {%highlight javascript%}
        $scope.checkProducts = function(){
          ProductsService.downloadProducts().then(function(response){
              $scope.newPackages = response
          })
        }
        {%endhighlight%}

the controller calls the productservice's downloadproducts method, to display the new products available to download

4. ProductsService.downloadProducts

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
        
