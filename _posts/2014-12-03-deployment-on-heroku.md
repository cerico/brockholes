---
layout: post
title: "Deployment on Heroku, Dokku, and DNS"
description: ""
category: 
tags: [deployment,heroku,dns]
---

#Heroku

A shorter post now, as we upload our nascent app to Heroku. Firstly, you can only deploy from the git master branch, so lets git merge our master branch into dev, fix any conflicts - though there werent any), switch back into master and merge dev

`➜  pennine git:(dev) git merge master`

`Already up-to-date.`

`➜  pennine git:(dev) git checkout master`

`Switched to branch 'master'`

`➜  pennine git:(master) git merge dev`

`Already up-to-date.`

ok lets create a heroku app

    ➜  pennine git:(master) heroku create pennine
    ➜  pennine git:(master) heroku addons:add heroku-postgresql
    ➜  pennine git:(master) heroku config:set aws_secret_access_key=weogunouwguqegnqegnqot
    ➜  pennine git:(master) heroku config:set aws_access_key_id=wougouwngouwng
    ➜  pennine git:(master) heroku config:set SCENICBUCKET=scenicone
    ➜  pennine git:(master) git push heroku master
    ➜  pennine git:(master) heroku run rake db:create
    ➜  pennine git:(master) heroku run rake db:migrate
    ➜  pennine git:(master) heroku run rake db:seed
    ➜  pennine git:(master) heroku restart
    ➜  pennine git:(master) heroku open

and out barely formed app is at http://pennine.herokuapp.com

#Dokku

Lets also deploy our app on Dokku, as an alternative. First we'll log into Digital Ocean, and create our Droplet, which we'll call apps.salterhebble.com (FQDN). Scroll down to Select Image, and then on the applications tab, select dokku, after you've set up your ssh keys

<img src="http://salterhebble.com/blogpics/do1.jpg">

<img src="http://salterhebble.com/blogpics/do2.jpg">

<img src="http://salterhebble.com/blogpics/do3.jpg">

it takes around 60 seconds to create a droplet, and if we have it a FQDN we should be able to browse to it to finish off setup, at http://apps.salterhebble.com (otherwise put the IP address in), lets select virtualhost naming, and finish setup

<img src="http://salterhebble.com/blogpics/do4.jpg">

we can now ssh into our droplet, and begin to configure


    ➜  pennine git:(master) ssh root@apps.salterhebble.com

      Welcome to Ubuntu 14.04.1 LTS (GNU/Linux 3.13.0-40-generic x86_64)

       * Documentation:  https://help.ubuntu.com/

      System information as of Thu Dec  4 07:25:37 EST 2014

      System load: 0.0               Memory usage: 4%   Processes:       53
      Usage of /:  9.6% of 29.40GB   Swap usage:   0%   Users logged in: 0

      Graph this data and manage this system at:
      https://landscape.canonical.com/

First lets check out VHOST file is as it should be

    root@apps:~# cat /home/dokku/VHOST
      apps.salterhebble.com
      
and start to configure or droplet

    apps.salterhebble.comroot@apps:~# cd /var/lib/dokku/plugins
    root@apps:/var/lib/dokku/plugins# git clone https://github.com/Kloadut/dokku-pg-plugin.git postgresq
      Cloning into 'postgresq'...
      remote: Counting objects: 261, done.
      remote: Total 261 (delta 0), reused 0 (delta 0)
      Receiving objects: 100% (261/261), 44.16 KiB | 0 bytes/s, done.
      Resolving deltas: 100% (140/140), done.
      Checking connectivity... done.
    root@apps:/var/lib/dokku/plugins# dokku plugins-install
    
 Now lets return to our local machine, and set up the droplet as a remote repository
 
     ➜  pennine git:(master) git remote add dokku dokku@apps.salterhebble.com:scenic
     ➜  pennine git:(master) git push dokku master 
    
 and once succesfully pushed, we can go back to our droplet, and set up a postgresl database
 
     root@apps:~# dokku postgresql:create scenicdb

         -----> PostgreSQL container created: postgresql/scenicdb 
      
     Url: 'postgres://root:blahblahblah@172.17.42.1:49165/db'
     
     
 Now lets link that to our app (you can do this via dokku postgresql:link app db, but ive had more luck doing it the following way)
 
     root@apps:~# dokku config:set  scenic DATABASE_URL=postgres://root:blahblahblah@172.17.42.1:49165/db
     

Now this WOULD have worked, except for the fact that we now have carrierwave/AWS in our app, so it throws up the following error

`ArgumentError: Missing required arguments: aws_access_key_id, aws_secret_access_key`

so we need to give dokku our AWS credentials, lets do that here. I had a lot of problems here, with region stuff being correct, so make sure this ties up properly with your AWS S3 bucket region location

    dokku config:set scenic aws_secret_access_key=sbjwugwougbwuoegbuwegb
    dokku config:set scenic aws_access_key_id=siubguwbguwbg
    dokku config:set scenic S3_REGION="s3-eu-west-1"
    dokku config:set scenic AWS_STORAGE_BUCKET_NAME=scenicone
    
and we can migrate and seed our droplet database

    root@apps:~# dokku run scenic bundle exec rake db:migrate
    root@apps:~# dokku run scenic bundle exec rake db:seed
    
#DNS
 
Now, we'd like this to play nicely with DNS so we don't have to use port numbers for our app. So here's my dns setup, lets go back to digital ocean and make sure thats set up correctly, here are the 3 important records

<img src="http://salterhebble.com/blogpics/dns101.jpg">

my main A record for the site, which is hosted on my original Digital Ocean droplet

<img src="http://salterhebble.com/blogpics/dns102.jpg">

my second A record, for the app server, which is another droplet with a different IP

<img src="http://salterhebble.com/blogpics/dns103.jpg">

and my CNAME record for the app server, which means all rails apps, will be appear under their name at apps.salterhebble.com, and we can see this by running the following

    root@apps:~# dokku url scenic
    http://scenic.apps.salterhebble.com
    
    
So our basic app is now running on both heroku and dokku, and while we're in the world of deployment and environment variables and the like, out next task is to setup signing in to our app, locally, on heroku and then on dokku, and then we can move back to working on the app itself. See you in a bit!
    
    







