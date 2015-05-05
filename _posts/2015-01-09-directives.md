---
layout: post
title: "directives"
description: ""
category: 
tags: [angularjs,directives]
summary: Moving various parts of the view into their own directives
---

Lets start by  making the standard 'hello world' directive

1) app/assets/javascripts/app/js.erb

{%highlight javascript%}
scenic.directive('helloWorld', function() {
  return {
      restrict: 'AE',
      replace: 'true',
      template: '<h3>Hi Dere!</h3>'
  };
});
{%endhighlight%}

2) app/assets/javascripts/templates/trail.html

{%highlight html%}
<div data-hello-world></div>
{%endhighlight%}

and it should show in our page, now lets move the content of the template to its own page, and reference it via templateURL

3) app/assets/javascripts/app/js.erb

{%highlight javascript%}
scenic.directive('helloworld', function() {
    return {
        restrict: 'AE',
        templateUrl: 'helloworld.html',
    };
});
{%endhighlight%}

4) app/assets/javascripts/templates/helloworld.html

{%highlight html%}
<h3>Hi Dere!</h3>
{%endhighlight%}

now, in the trail view, we can move the sidebar, the mainphoto, the map, the morephotos and the dropzone into their own directives