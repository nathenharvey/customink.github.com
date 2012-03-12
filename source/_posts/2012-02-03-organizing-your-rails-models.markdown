---
layout: post
title: "Organizing Your Rails Models"
date: 2012-02-03 15:38
comments: true
categories: 
  - rails
  - ruby
author: karle-durante
published: true
---

Like people, applications start out small.  Unlike people, applications do not always have a predictable growth pattern.  Sometimes they grow really big, and sometimes not at all.  When applications grow large, organization becomes important because it is no longer possible to remember every detail about your application without consulting the source code. 

"We use Rails, the convention tells us how to organize our code".  Yes, but having 50 or more files in your app/models directory is hardly being organized.  

A simple thing I like to do is group related models into folders.  This allows you to organize the related models of a domain into a single location while any shared or stand alone models simply remain in the root directory.  For instance, my directory structure may look something like:

```
app
|_models
   |_address.rb
   |_orders
      |_order.rb
      |_item.rb
      |_shipping_detail.rb
```

And to make sure Rails can find all of my models, I need to update config/application.rb file as such:

```ruby
config.autoload_paths += ['app/models/**"]
```

This allows developers to quickly see what models make up an order and what models either stand on their own, or are shared across multiple domains.  And for a large application, you are now able to quickly summarize the high level business objects that make up your system.
