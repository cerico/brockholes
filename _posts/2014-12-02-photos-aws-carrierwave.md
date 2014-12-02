---
layout: post
title: "Photos, AWS, Carrierwave"
description: ""
category: 
tags: [AWS,S3,Fog,carrierwave,rails,html]
---
#AWS

To get the photos to work on our site we need somewhere to host them, so we'll sign up with AWS, and go to the S3 page

[https://console.aws.amazon.com/s3/home?region=us-west-2](http://)

go to security credentials

<img src="http://salterhebble.com/blogpics/az.jpg">

create a new user

<img src="http://salterhebble.com/blogpics/az2.jpg">

and get the access key ID and secret access key

<img src="http://salterhebble.com/blogpics/az3.jpg">


make a note of these two access keys, we need to put them in out .bashrc or .zshrc (for linux/mac and, er, wherever it is windows people put things )

`export aws_secret_access_key="khebfhwbgiwebgwbgkewbgwbguiwrbguwrebg"`

`export aws_access_key_id="iwuebgiwebguiwebg"`

Now we need to create a bucket

<img src="http://salterhebble.com/blogpics/bucket.jpg">

and also put this in our .bashrc or .zshrc

`export SCENICBUCKET=scenicone`

#Carrierwave

Now we need the carrierwave gem, so lets add it to our Gemfile, along with RMagick, and Fog

`gem 'carrierwave'`
 
`gem 'rmagick', :require => 'RMagick'`

`gem 'fog', '~> 1.3.1'`

and run bundle to install

`➜  pennine git:(dev) bundle`

Lets generate our uploader

`➜  pennine git:(dev) rails g uploader TrailPhoto`


which create the following file

  app/uploaders/trail_photo_uploader.rb
  
  lets open that, and uncomment the following line, so we have RMagick support 
  
  `# include CarrierWave::RMagick`
  
  and add the following two lines
  
  `include CarrierWave::MimeTypes`
    
  `process :set_content_type`
  
  now we can choose what size images, we want carrierwave to make for us. As these images are going to appear in different sizes on different pages, ive given myself a bit of leeway, and added the following range
  
     version :thumb do
       process :resize_to_fill => [50, 50]
     end

     version :detail do
       process :resize_to_fill => [300, 300]
     end

     version :medium do
       process :resize_to_fit => [800, 800]
     end

     version :trailpage do
       process :resize_to_fit => [840, 551]
     end
   
     version :frontpage do
       process :resize_to_fill => [400, 440]
     end
     
     version :wide do
       process :resize_to_fit => [1200, 600]
     end

     version :large do
       process :resize_to_fit => [1600, 1600]
     end


and finally, uncomment the fog line, and comment the file line, as we wont be hosting images locally, so it should look like this

`  #storage :file`

   `storage :fog`

Now, we'll need a file for the config options, so lets create that here

`➜  pennine git:(dev) touch  config/initializers/carrierwave.rb`

and add the following block

    CarrierWave.configure do |config|
      config.fog_credentials = {
        :provider  => 'AWS',  # required
        :aws_access_key_id  => ENV['aws_access_key_id'],  # required
        :aws_secret_access_key => ENV['aws_secret_access_key'],  # required
        :region  => 'eu-west-1',  # optional, defaults to 'us-east-1'
      }
      config.fog_directory  = ENV['SCENICBUCKET']  # required
      config.fog_public  = true  # optional, defaults to true
    end
    
which tells carrierwave we're using the AWS and that out credentials and bucket are stored in our .bashrc (or .zshrc) (rather than having our keys displayed here)
    
 Now we can scaffold our Photo model
 
 `➜  pennine git:(dev) rails g scaffold Photo user_id:integer trail_id:integer name image`
 
Where photo belongs_to trail, and also to user, so lets add that into app/models/photo.rb, and also configure it to use our new trail_photo_uploader

    belongs_to :user
    belongs_to :trail

    mount_uploader :image, TrailPhotoUploader
    
 and lets edit our app/models/trail.rb accordingly, to reflect that it has_many photos
 
    
    has_many :bookmarks
    has_many :photos
    has_many :comments
    belongs_to :user
    
    accepts_nested_attributes_for :photos, allow_destroy: true
    
 Now lets seed the database with some photos, i've put the initial photos i want to use into public, and then add the following into the db/seeds.rb, underneath where we added the trails seed data earlier 
 
     t1.photos.create(image: File.open(File.join(Rails.root.join('public'), 'hike2.jpg')))
     t2.photos.create(image: File.open(File.join(Rails.root.join('public'), 'kieldersnow.jpg')))
     t2.photos.create(image: File.open(File.join(Rails.root.join('public'), 'kielder.jpg')))
     t2.photos.create(image: File.open(File.join(Rails.root.join('public'), 'kielder2.jpg')))
     t2.photos.create(image: File.open(File.join(Rails.root.join('public'), 'kielder3.jpg')))
     t3.photos.create(image: File.open(File.join(Rails.root.join('public'), 'ullswater.jpg')))
     t4.photos.create(image: File.open(File.join(Rails.root.join('public'), 'bowland.jpg')))
     t5.photos.create(image: File.open(File.join(Rails.root.join('public'), 'todmorden.jpg')))


and so on (truncated for brevity)

now lets seed, and we can see that our photos have succesfully gone to AWS, by going to the photos page

`➜  pennine git:(dev) ✗ rake db:drop db:create  db:migrate  db:seed`


<img src="http://salterhebble.com/blogpics/a6.jpg">

but this isnt where we want them, we want them on our trails page, so lets go to our app/views/trails/index.html.erb and replace the hardcoded image with a ruby tag

Lets replace 

`<img src="https://calderscenic.s3.amazonaws.com/uploads/photo/image/2/frontpage_tod1.jpg">`

with

`<%= image_tag(trail.photos.first.image) %>`

and have a look in the browser

<img src="http://salterhebble.com/blogpics/mismatchedimages.jpg">

but its trying to cram in different sized images into the div, now this is where carrierwave helps out, with the fact that we configured it to create differently sized images, for use in different parts of the site, so lets change 

`<%= image_tag(trail.photos.first.image) %>`

to

`<%= image_tag(trail.photos.first.image.frontpage.url) %>`

which we had previously configured to be 400x440 in our trail_photo_uploader.rb

and as that image is also a link lets change it again, from 

`<a href="trail.html"><%= image_tag(trail.photos.first.image.frontpage.url) %></a>`

to 

`<%= link_to image_tag(trail.photos.first.image.frontpage.url), trail %>`


And now out trails index page looks is starting to take shape. Final task for now, is to change the background to a rails tag also, so lets go to app/views/layouts/application.html.erb

and change

`<img src="./j14.jpg">`

to 

`<%= image_tag("j14.jpg") %>`


 
 
 
 

