---
layout: post
title: "Simple Garbage Collection Tuning for Rails"
date: 2012-03-16 08:07
author: jake-vanderdray
comments: true
published: false
categories: 
  - apache
  - garbage collection
  - gdb.rb
  - passenger
  - performance
  - ruby
  - ruby enterprise
---

Ruby is known for being bad at garbage collection.  The truth is that the default GC settings aren't very good for a Rails application so if you run a Rails app you really should do some tuning (this requires either Ruby Enterprise or Ruby 1.9.2).  Here's a streamlined process for getting started:

### Get a Baseline

Turn on collecting GC stats for New Relic (of course you're using [New Relic](http://newrelic.com/)).  You want to know what you're fixing and this will probably show you that about &#8531; of the "Ruby" portion of your app response time is really garbage collection.  Just add the following line to your `environment.rb` file:

`GC.enable_stats if defined?(GC) && GC.respond_to?(:enable_stats)`

### Examine the Heap

Once you've gathered enough data in NewRelic to be able to see a change, you'll want to see what the heap looks like in one of your passenger threads.  

* Take one of your app servers out of production
* Install gdb.rb:

`sudo gem install gdb.rb`

* Use `sudo passenger-status` to find a thread that has handled enough requests to be pretty well warmed up and note its PID.
* Connect to the passenger thread with gdb.rb:

`sudo gdb.rb <pid>`

* Get gdb.rb to print out the stats about your objects:

`ruby objects`

You're looking for a section that looks like this:

    HEAPS            9
      SLOTS      3061241
      LIVE       1457106 (47.60%)
      FREE       1604135 (52.40%)

We're going to assume that "LIVE" number is representative of how many slots you normally use up.  Round that up to something sensible like 1,500,000.  Now do the math like this:

    RUBY_HEAP_MIN_SLOTS=1800000          # Slots Live + 20%
    RUBY_HEAP_FREE_MIN=18000             # 1% of HEAP_MIN_SLOTS
    RUBY_HEAP_SLOTS_INCREMENT=144000     # 8% of HEAP_MIN_SLOTS
    RUBY_HEAP_SLOTS_GROWTH_FACTOR=1
    RUBY_GC_MALLOC_LIMIT=60000000

I know there's no explanation for those last two settings, but I haven't really explained the math behind the other numbers either.  This is meant to be a good starting point.  Its customized for your app to some degree, but with some assumptions.

## Wrap Your Ruby


Now create a wrapper script that sets these variables in the environment before calling ruby.  I'm going to assume you put it in `/usr/local/bin` and call it `ruby_tuned`.  The file should look like this (make sure you adjust for the path to ruby on your system):

    #!/bin/bash

    export RUBY_HEAP_MIN_SLOTS=1800000
    export RUBY_HEAP_FREE_MIN=18000
    export RUBY_HEAP_SLOTS_INCREMENT=144000
    export RUBY_HEAP_SLOTS_GROWTH_FACTOR=1
    export RUBY_GC_MALLOC_LIMIT=60000000

    exec "/usr/local/bin/ruby" "$@"

## Update Passenger


Have passenger use your `ruby_tuned` wrapper instead of calling ruby directly by updating `passenger.conf` (look in `/etc/apache2/mods-enabled` on Ubuntu).  You'll want it to look like this:

    PassengerRoot /usr/local/lib/ruby/gems/1.8/gems/passenger-3.0.11
    PassengerRuby /usr/local/bin/ruby_tuned
    
Now restart apache, add the server back into production and check NewRelic to see how you did.

## What We Got

<img src="/images/Response_Time_GC.jpg">

The graph above is from New Relic as I rolled the changes out one server at a time.  When we applied these changes to our first app we saw:

* Time spent in GC drop from ~35ms per request to ~10ms
* CPU usage drop almost in half
* A slight increase in memory used

## Where Those Numbers Actually Came From

To understand these settings and what they do checkout:

* [This Presentation from Joe Damato](http://www.viddler.com/v/87ae120a)
* [This Post from Chris Heald](http://www.coffeepowered.net/2009/06/13/fine-tuning-your-garbage-collector/) (he adds a gem to his app instead of using gdb.rb) 
* [The Ruby Enterprise GC documentation](http://www.rubyenterpriseedition.com/documentation.html#_garbage_collector_performance_tuning)
