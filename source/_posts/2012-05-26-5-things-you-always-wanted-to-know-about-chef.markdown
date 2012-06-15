---
layout: post
title: "5 Things You Always Wanted to Know About Chef"
date: 2012-05-26 21:51
comments: true
published: true
author: nathen-harvey
categories: 
  - ChefConf
  - ChefConf2012
  - chef
  - conference
  - devops
  - rails
  - sysadmin
  - web operations
---
When I first started working with Chef, there were a couple of areas that I knew were going to be really awesome and helpful but I wasn't sure how to get started with them.  In this presentation, I'll provide a quick introduction to five things you've always wanted to know about Chef but were afraid to ask.  

I gave this presentation at [#ChefConf 2012](http://chefconf.opscode.com).

Level-up your Chef skills by learning about these areas of Chef: 

<!-- more -->

* **Attribute Precedence** - Role, environment, cookbook, data bag? Which attribute value will be used in my chef run? 
* **Encrypted Databags** - Chef 0.10 brought us encrypted databags. We'll look at how to create and use databags and how to keep them up-to-date in your repository. 
* **LWRP** - What is a LWRP? How and why do you create one? We'll look at a couple of sample LWRPs and learn how to build a simple one. 
* **Error Handlers** - Demystify exception and report handlers by writing a simple one and seeing examples of how they work in the wild. 
* **Capistrano and Chef** - Take a quick look at why and how to integrate Chef search into your Capistrano configuration to make deploying your Rails apps even easier.

One thing I didn't mention in the presentation was how to use the data from the encrypted data bag.  I've updated the slides to include this info but it doesn't appear in the video.  In any case, here's a quick demo of how you might use it:

``` ruby
creds = Chef::EncryptedDataBagItem.load("db", "creds")
env_db_creds = db_creds[node["rails_env"]]

template "#{app_dir}/shared/config/database.yml" do
  source "database.yml.erb"
  variables(
    :rails_env => node["rails_env"],
    :username => env_db_creds["username"],
    :password => env_db_creds["password"]
  )
end
```

### Video

<iframe width="560" height="315" src="http://www.youtube.com/embed/uREL4FFPddo" frameborder="0" allowfullscreen></iframe>

### Slides

<script async class="speakerdeck-embed" data-id="4fb532f2850667001f0008f8" data-ratio="1.2945638432364097" src="//speakerdeck.com/assets/embed.js"></script>
