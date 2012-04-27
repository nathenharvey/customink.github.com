---
layout: post
title: "Critical Oversight: Indexes"
date: 2012-04-27 11:48
comments: true
categories: 
  - process
  - indexes
  - rails
author: karle-durante
published: true
---

One of the most common production issues I run into are missing indexes.  The other day I got to thinking that they are usually missing because of the evolution of the software.  

We might use some rails generators to prototype some basic functionality.  Then we'll iterate over a set of stories incorporating new behavior.  Maybe we'll do some refactoring, scrap some features, pull out some dead code and "harden" some areas we've identified as brittle.  But we almost never analyze the "data model" before we deploy.

Have we considered our data access patterns?  Did we create foreign keys to enforce data integrity?  Do we have any idea how big these tables are going to grow?  We almost certainly don't need to shard them…do we?

No, we almost never do this.  

Instead we race to ship.  "[Deploy early, deploy often](http://www.customink.com/lab/?cid=jub0-000p-fxs7#shared)" is our motto, and we love it.  Deploying code is awesome, it means people are going to use it.  People using our code makes us happy because it means we didn't waste our time today.  We did something real that people got to use.

A few months down the road comes the tipping point.  One of your tables amasses a few hundred thousand rows.  Your pages start to take seconds to load because your queries take seconds to run.  Your database connections are tied up, and when your site gets enough traffic, things start to fall over.  

This literally just happened to us.  Again.  We missed one little index on one little foreign key in one little table.  And then one of our database servers stopped responding.   Spiked CPU, connections maxed out, alerts firing, then fail over.

While Rails makes it really easy to create models without even thinking about the database, Rails also makes it very easy to deal with the database.  In James Edward Gray's talk [10 Things You Didn't Know Rails Could Do](http://speakerdeck.com/u/jeg2/p/10-things-you-didnt-know-rails-could-do?utm_source=rubyweekly&utm_medium=email) he shows you how to use rails migration generators to create your table AND index your fields.

If you don't like generators, you can simply use the 'add_index' method in the migration itself:

```ruby
class CreateFoo < ActiveRecord::Migration
  self.up
    create_table :foos do |t|
      t.integer :foreign_key
      t.string  :other_valuable_data
    end
    
    add_index(:foos, :foreign_key, :name => 'foos_foreign_key')
  end
end
```
Of course, if you don't think to add the index when you created the table, you can always create a migration just to add the index.  The key is adding a checkpoint to your development process in which you analyze your data structures for completeness.  Adding this checkpoint gives you a chance to add any missing database constructs before it's too late.