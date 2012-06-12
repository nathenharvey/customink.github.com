---
layout: post
title: "Introducing Stoplight: Greenscreen 2.0"
date: 2012-06-12 17:48
comments: true
author: nathen-harvey
categories:
  - devops
  - build monitoring
  - greenscreen
  - jenkins
  - travis-ci
  - testing
---
<a href="/images/stoplight.png"><img src="/images/stoplight.png" width="300" style="float:right; margin:0 0 15px 15px;" /></a>
At CustomInk, we use a variety of tools to monitor the status of our builds. One such tool was [Greenscreen](https://github.com/customink-webops/greenscreen). In fact, we even wrote [a blog post about how we use Greenscreen at CustomInk](http://technology.customink.com/blog/2012/01/02/green-screen/) not too long ago.

One of the biggest problems with Greenscreen was its extensibility. By default, Greenscreen only works with Hudson and Jenkins servers. With [Travis CI](http://travis-ci.org) becoming quite popular in the open-source community, Greenscreen needed a major upgrade. Furthermore, Greenscreen was not very extensible.

After some significant refactoring, Greenscreen evolved into [Stoplight](https://github.com/customink/stoplight)...

<!-- more -->

Significant Improvements
------------------------
There are a number of improvements in Stoplight. These are the most critical or useful to the end-user:

- Support for multiple (any) continuous integration server
- Highly configurable yaml files
- Cross-browser beautiful UI
- More informative build statuses
- Extensibility
- Usability
- Full test suite


Refactoring
-----------
Originally, I was just going to add Travis CI support to Greenscreen. However, I quickly asked myself, "why stop at Travis CI?"; we should allow developers to connect Greenscreen to any continuous integration server. After cleaning up the code a bit, I introduced the concept of a `Provider`. Simply put, a `Provider` is an abstract Ruby class that maps server-data into Greenscreen data. It looks like this:

```ruby
# Provider is an abstract class that all providers inherit from. It requires that a specified format be returned. This way, stoplight
# doesn't care who it's talking to, as long as it guarantees certain information.
module Stoplight::Providers
  class Provider
    attr_reader :options, :response

    # Initializes a hash `@options` of default options
    def initialize(options = {})
      ...
    end

    # `projects` must return an array of Stoplight::Project
    # see Stoplight::Project for more information on the spec
    def projects
      ...
    end
  end
end
```

On the front-end, now we don't have to worry about parsing different server responses; we know that any provider will respond to the `Provider#projects` instance method. This makes refactoring our front-end code much easier.

The `Stoplight::Project` clearly defines a schema and method-set that must be adhered to. In the front-end, we can simply call `.projects` on any provider and know with 100% certainty that the given objects respond to a certain set of methods. Those methods are defined in the `Stoplight::Project` class:

```ruby
module Stoplight
  class Project
    attr_accessor :name, :build_url, :last_build_id, :last_build_time, :last_build_status, :current_status

    # Initialize (new) takes in a hash of options in the following format:
    #
    # {
    #   :name => 'my_project',
    #   :build_url => 'http://ci.jenkins.org/job/my_project',
    #   :web_url => 'http://github.com/username/my_project', # optional
    #   :last_build_id => '7',
    #   :last_build_time => '2012-05-24T03:19:53Z',
    #   :last_build_status => 0,
    #   :current_status => 1,
    # }
    #
    # - `name` - the name of this project
    # - `build_url` - the url where the build came from
    # - `build_id` - the unique build_id for this project
    # - `last_build_time` - last successful build
    # - `last_build_status` - integer representing the exit code of the last build:
    #   - -1: unknown
    #   -  0: passed (success)
    #   -  1: failed (error, failure)
    # - `current_status` - the current status of the build:
    #   - -1: unknwon
    #   -  0: done (sleeping, waiting)
    #   -  1: building (building, working, compiling)
    def initialize(options = {})
      ...
    end
  end
end
```
This set of simple instructions tells a provider how it must format data. Essentially this makes `Provider` a micro-data-mapper, massaging data from remote APIs into a standard format. Stoplight then uses that standard format to create a unified user experience.

### Adding Tests
Since Stoplight is destined to become an open-source project, it needs a comprehensive test suite. Furthermore, during our refactoring, it's important that we don't break existing functionality. As such, I added a full test suite with RSpec. To make development easier, I also use Spork, Guard, and Growl-Ruby to automatically run tests in the background while I'm coding. This makes TDD much more exciting.

### Foreman
With all those dependencies, plus running the server, it only made sense to use [Foreman](https://github.com/ddollar/foreman) to manage all those processes. What used to be (in three different terminal tabs):

    bundle exec shotgun -p 4567
    bundle exec compass watch -c config/compass.rb
    bundle exec guard

simply became:

    bundle exec foreman start

with a tiny `Procfile`:

```
web:      shotgun -p 4567
compass:  compass watch -c config/compass.rb
guard:    guard
```


### Compass
Under the hood, all the styles for Stoplight are generated by a framework called [Compass](http://compass-style.org). Compass allows us to leverage the power of SCSS and easily create a cross-browser compliant application. Stoplight uses Compass and SCSS for all it's styles. This ensures a consistent user experience.


### ABAP Text
ABAP stands for "As Big As Possible". A tiny snippet of jQuery dynamically sizes text to fit in its bounds. Whether you're displaying on a 60" LED TV or a 13" Monitor, the text will be as big and clear as possible.


### Adding Travis
Brace yourself for a meta-moment. Since I wanted Stoplight to become a popular open-source project, I added the build on Travis CI. With [Stoplight on Travis CI](http://travis-ci.org/#!/customink/stoplight) we can use Stoplight to monitor the build status of Stoplight.


Conclusion
----------
There you have it! What was once a very isolated and less-than-configurable tool is now highly extensible and open to contributions by the community. If you are currently using Greenscreen or another build monitoring alternative, give Stoplight a try.


I Want it!
----------
Stoplight is available for [download and forking on github](https://github.com/customink/stoplight). Pull requests are greatly welcome! There is also a highly-configurable [chef cookbook for installing Stoplight](https://github.com/customink-webops/stoplight) on Apache and Passenger on your own servers.
