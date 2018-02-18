---
layout: post
title: "Octopress blog site setup"
date: 2015-12-12 11:53:18 +0530
comments: true
categories: [ Octopress ]
---
I am using octopress quite some time for my technical blog and the octopress framework is interesting but sometimes I felt difficult to setup while working in different machines.Because I need to setup the entire source code to blog a single post. So just thought to document the steps.

<!-- more -->
## Setup for fresh site
```
git clone git://github.com/imathis/octopress.git octopress
cd octopress
# setup for installing the dependencies
gem install bundler
rbenv rehash    
bundle install
# Install the default theme
rake install
```
Now you are set to for happy blogging. Of course first you are gonna modify _config.yml file then you can start creating post, page as below
```
rake new_post[post_title]
rake new_page[page_title]
```
## Setup for existing site
```
git clone https://github.com/<username>/<username>.github.io.git
cd <username>.github.io.git
git checkout source
mkdir _deploy
cd _deploy
git init
git remote add origin https://github.com/<username>/<username>.github.io.git
git pull origin master
cd ..
```
Now everything is set to blog as usual.
## Deploy in Githup Pages
There are two tasks in this. One is deploying your generated blog site in 'master' branch and second is backup the site source in git 'source' branch.

###Deploy the blog site:
```
rake setup_github_pages
rake generate
#rake preview # to verify your site in local machines
rake deploy
```
###Backup the source :
```
git add .
git commit -m 'commit details'
git push origin source
```
This needs to be done everytime you deploy your site which helps you to quickly start blogging from any machine if have ineternet.
