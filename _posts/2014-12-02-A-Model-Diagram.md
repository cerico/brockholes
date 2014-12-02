---
layout: post
title: "Model Diagram"
description: ""
category: 
tags: [planning]
---
Ive used railroady to create a Model Diagram, so there's some extra stuff in here but I prefer this to other tools for creating diagrams

<img src="http://salterhebble.com/s3/models.svg">

We'll build this incrementally so some parts may well not feature. The main models are Trails, Photos, Users and Bookmarks (Relationships and Comments will come later). The user model has a lot of Devise stuff in there, which we can ignore for now. I'll run through each  model.

1) Trails

A trail has_many photos, bookmarks and comments, it also belongs to a user (who is the trail uploader)

2) Users

A user has_many trails (through bookmarks), and also has_many comments and bookmarks, a user  will also has_many followed_users, through a relationships model

3) Bookmarks

A bookmark belongs_to a user, and also to a trail. This model will be used for favouriting, completing and ratings (but not for adding - because a trail can have many users who have completed, rated or favourited it, but only one user who uploaded it)

4) Photos

A photo belongs_to a trail, and also to a user

5) Comments belong to a trail and a user (and could probably move into the bookmarks model instead of being their own)

6) The relationships model is what allows "a user to has_many users"

We'll go into proper detail on each one as they get made, for now its just to visualize the functioning of the site