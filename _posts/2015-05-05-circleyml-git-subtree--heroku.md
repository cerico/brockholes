---
layout: post
title: "circle.yml, git subtree & heroku"
description: ""
summary: I haven't got too much into testing and continuous integration yet, but I wanted to have a basic test that kept the angular application and the rails server in line with each other, before starting to write more granular tests for eac"
category: 
tags: []
---
I haven't got too much into testing and continuous integration yet, but I wanted to have a basic test that kept the angular application and the rails server in line with each other, before starting to write more granular tests for each. We've merged both respositories into one, with the angular application in app, and the rails server in server, but of course git push no longer works as the rails server isnt at the top level, which is where git subtree comes in

So our situation is that we have a live heroku, a UAT heroku, and a staging heroku,  and we want to be able to push to each from a continuous integration server, for which we chose <a href="http://circleci.com">circleci</a>. Our initial problem is when we merge staging into UAT, we need to make sure that the heroku it points to stays as UAT and doesnt get updated to the staging heroku. The flow is as follows, we push to github, and circleci picks up the change and immediately attempts to push that branch to heroku. Here is a basic version of our circle.yml

{%highlight javascript%}
deployment:
  staging:
    branch: staging
    commands:
      - git subtree push --prefix server git@heroku.com:staging-regulated.git master

  uat:
    branch: uat
    commands:
      - git subtree push --prefix server git@heroku.com:uat-regulated.git master

  master:
    branch: master
    commands:
      - git subtree push --prefix server git@heroku.com:regulatedsales.git master


test:
  override:
    - ./test.sh
    {%endhighlight%}
    
This detects when any of the 3 branches has been pushed to on github, and then runs the subtree command. We can see that each branch goes to its own heroku, but what might be confusing is 'master' at the end of each command, this is because we are pushing to the master branch of each heroku. And we're posting the server subtree, as that is the head of the rails application. The way i set this up first was to push each branch locally to heroku, to make sure all the environment variables were correct, and then removed the remotes

So lets look at the test.sh

I wanted a basic test, that would live in all the branches and just makes sure the angular app is pointing to the correct heroku instance. It doesnt stop us posting to github, but it fails the test and doesnt post to heroku

{%highlight ruby%}
BRANCH=`git branch | sed -n -e 's/^\* \(.*\)/\1/p'`
if [ $BRANCH ==  "master" ];
  then grep "live.herokuapp.com" app/www/js/app.js;
elif [ $BRANCH ==  "uat" ];
  then grep "uat.herokuapp.com" app/www/js/app.js;
elif [ $BRANCH ==  "staging" ];
  then grep "staging.herokuapp.com" app/www/js/app.js;
else false;
fi
{%endhighlight%}

firstly we set up a variable which is the current branch, and then we simply grep to make sure that the angular app is pointing to the correct heroku, and if not we return false and the test fails, and the server isnt deployd to heroku

