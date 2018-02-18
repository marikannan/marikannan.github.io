---
layout: post
title: "Deploy meteor app in heroku"
date: 2016-05-08 15:24:10 +0530
comments: true
categories: [ Heroku, Meteor ]
---
In this post, will give quick guide to deploy an meteor application into heroku web hosting.

Assuming that you already have following prerequistes,

1. Meteor js application is running in local system ( https://www.meteor.com )
2. Install heroku in local system ( https://toolbelt.heroku.com )
3. Heroku account ( verified ) in www.heroku.com
4. git installed in local system

Here is the list of steps to deploy meteor application in heroku.

*  Go to meteor application's root directory

``` bash  
cd sample-meteor-app
```

*  Initialize git for the appln directory.

``` bash  
git init
git add .
git commit -am "Initial Commit"
```

*  Creating a sample heroku application.

``` bash 
heroku create sample-app
```

*  Setting up the build packs which tells heroku how to setup and build the heroku container.

``` bash
heroku buildpacks:set https://github.com/AdmitHub/meteor-buildpack-horse.git
```

*  Installing mongodb addons to support for our meteor application.

``` bash
heroku addons:create mongolab
```

*  Setting the application url

``` bash
heroku config:set ROOT_URL=https://sample-app.herokuapp.com
```

*  Deploying the heroku application in heroku repository.

``` bash
git push heroku master
```

