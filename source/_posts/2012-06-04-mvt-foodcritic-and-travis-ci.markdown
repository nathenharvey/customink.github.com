---
layout: post
title: "MVT:  Foodcritic and Travis CI"
date: 2012-06-04 13:45
author: nathen-harvey
comments: true
published: true
categories: 
  - chef
  - chefconf
  - chefconf2012
  - customink
  - devops
  - foodcritic
  - opschef
  - opscode
  - process
  - test driven development
  - testing
  - travisci
  - tutorial
  - web operations
  - webops
---
One of the big themes that emerged during [#ChefConf](http://chefconf.opscode.com/) was that we should be testing our infrastructure code.  Software engineers have been practicing test-driven development, behavior-driven development, continuous integration, and many other testing-related practices for a long time.  It's becoming more important for the infrastructure engineers to learn from and apply these practices to our day-to-day workflow.  When it comes to testing Chef-driven infrastructure automation, there are a number of tools and practices that are starting to emerge.  In this article I'll look at a "minimum viable testing" (MVT) approach to this problem using [Foodcritic](http://acrmp.github.com/foodcritic/) and [Travis CI](http://travis-ci.org/).  [Follow the steps in this article](http://technology.customink.com/blog/2012/06/04/mvt-foodcritic-and-travis-ci/#steps) to get your public cookbooks tested after every `git push`.

### Testing with Chef

The idea of building automated tests for your infrastructure code has been getting a lot of traction lately.  When it comes to [Chef](http://www.opscode.com/chef/), many tools are starting to emerge.

<!-- more -->

The first tool in this area to get any significant traction, that I know of, was [cucumber-chef](http://www.cucumber-chef.org/).  I first learned of this tool when I saw a pre-release copy of [Test-Driven Infrastructure with Chef](http://shop.oreilly.com/product/0636920020042.do) at the O'Reilly booth at [Velocity Conf 2011](http://velocityconf.com/velocity2011).  [Stephen Nelson-Smith](http://twitter.com/lordcope), the book's author and framework's lead developer, proposes an outside-in approach to testing where your tests can also act as monitors that look after the health of your infrastructure.  I like the idea of this approach and feel it makes a lot of sense in a greenfield environment.  One benefit of this approach is that it blurs the line between testing and monitoring.  You can easily hook-up your monitoring system to your cucumber tests.

[ChefSpec](https://github.com/acrmp/chefspec) is another tool for testing your Chef code.  It is a gem that makes it easy to write [RSpec](http://rspec.info/) examples for Chef cookbooks.  This style of testing allows you to execute your tests without needing to converge the node that your tests are running on.  In other words, you can execute your tests without needing to provision a server.  One huge appeal to this style of testing is that the feedback loop is very small.  You'll get feedback about your cookbook changes within seconds or a very few minutes of saving your changes.

[Minitest Chef Handler](https://github.com/calavera/minitest-chef-handler) is yet another tool for testing with Chef.  This runs a suite of [minitest](https://github.com/seattlerb/minitest) tests as a report handler in your Chef-managed nodes.  As you may know, report handlers are run at the end of each [chef run, or convergence](http://wiki.opscode.com/display/chef/Anatomy+of+a+Chef+Run).

### Testing at ChefConf

At the inaugural [#ChefConf](http://chefconf.opscode.com) there were many sessions that included information about many companies' approach to testing.  Here's a quick list of some of the sessions:

* [Food Fight Show Episode #10 - TESTALLTHETHINGS](http://www.foodfightshow.org/2012/04/episode-10-testallthethings-testing.html) -- This wasn't actually part of #ChefConf but is 'required listening' for anyone interested in learning more about this space.

* [#ChefConf Pre-event Hackday: TEST ALL THE THINGS!!!](http://chefconf2012.sched.org/event/bfe13edac99e2b4d8582f0cd1005ee73?iframe=no&w=700&sidebar=no&bg=no)

* [NTP Cookbook with tests](https://github.com/atomic-penguin/ntp) - tests were added to this cookbook as part of the hackday event.

* [Test-driven Development for Chef Practitioners](http://www.youtube.com/watch?v=o2e0aZUAVGw) (video)

* [Test Driven Development Roundtable](http://www.youtube.com/watch?v=dPaYfAIvqxw) (video)

<!--more-->

### Foodcritic

[Foodcritic](http://acrmp.github.com/foodcritic/) is a lint tool for your Chef cookbooks.

{% blockquote http://acrmp.github.com/foodcritic/ %}

Foodcritic has two goals:

* To make it easier to flag problems in your Chef cookbooks that will cause Chef to blow up when you attempt to converge. This is about faster feedback. If you automate checks for common problems you can save a lot of time.

* To encourage discussion within the Chef community on the more subjective stuff - what does a good cookbook look like? Opscode have avoided being overly prescriptive which by and large I think is a good thing. Having a set of rules to base discussion on helps drive out what we as a community think is good style.
{% endblockquote %}

#### Why start with Foodcritic?

Given the plethora of options available, why should you start with Foodcritic?  Well, you have to start somewhere.  We felt Foodcritic was a good choice because it was easy to get started with, the tests ran quickly, and we are working under the assumption that once we started some automated testing, we'll start layering on more and more pieces as we go.  After some initial experiments, we found that we could get Foodcritic looking after our each cookbook in a matter of minutes and local tests running in seconds.

The pseudo-converge approaches (like ChefSpec) initially feel like we'll need to do a lot of mocking that will take some time to get correct.  The post-converge approaches (like cucumber-chef and minitest) will take longer to run and are a bit more complex.

One benefit of the post-converge approach is the ability to use your tests as health monitors.  We already have monitoring in place and use it as an indicator that a node is fully provisioned.  We call this "monitor-driven development."  Given that, it was better for us to get started with something that runs without requiring a full converge.  Foodcritic fit the bill quite nicely.

### Travis CI

Travis CI is:

{% blockquote http://about.travis-ci.org/docs/ %}
A hosted continuous integration service for the open source community.
{% endblockquote %}

Using Travis CI in conjunction with Foodcritic, we'd have a basic automated test foundation to build on.

### Automated Foodcritic tests with Travis CI <a name="steps"></a>

Using Foodcritic and Travis CI, you can quickly set-up a "minimum viable testing" (MVT) environment.  The idea is that once you have some sort of tests running against your cookbooks, you'll want to add more and doing so will be easy.  Let's look at how to add Foodcritic and Travis CI to your cookbook workflow.

#### Initial set-up

Follow these steps to get everything set-up and ready for your first tests:

1. `gem install foodcritic`
1.  Go to [Travis CI](http://travis-ci.org/) and follow the Sign In link at the top.
1.  Activate the GitHub Service Hook for your cookbook's repository from your TravisCI profile page.  Each of your cookbooks has its own repository, right?!

#### Configure your project

The next step is to add a .travis.yml file to your project.

``` ruby .travis.yml
language: ruby
gemfile:
   - test/support/Gemfile
rvm:
  - 1.9.2
  - 1.9.3
script: bundle exec rake foodcritic
```

This file tells Travis CI how to build your project.  We've specified the language (ruby) and the versions of ruby to use when testing this cookbook (1.9.2 and 1.9.3).  We've also specified a Gemfile and script to execute when testing this project.  Let's add a Gemfile to a new directory in our cookbook, `test/support`.

``` sh
mkdir -p test/support
touch test/support/Gemfile
```

Our Gemfile is pretty simple, just include `rake` and `foodcritic`.


``` ruby Gemfile
source "https://rubygems.org"

gem 'rake'
gem 'foodcritic'
```

Finally, we'll need to add a Rake file that will be run each time Travis builds our project.

``` ruby Rakefile
#!/usr/bin/env rake

desc "Runs foodcritic linter"
task :foodcritic do
  if Gem::Version.new("1.9.2") <= Gem::Version.new(RUBY_VERSION.dup)
    sandbox = File.join(File.dirname(__FILE__), %w{tmp foodcritic cookbook})
    prepare_foodcritic_sandbox(sandbox)

    sh "foodcritic --epic-fail any #{File.dirname(sandbox)}"
  else
    puts "WARN: foodcritic run is skipped as Ruby #{RUBY_VERSION} is < 1.9.2."
  end
end

task :default => 'foodcritic'

private

def prepare_foodcritic_sandbox(sandbox)
  files = %w{*.md *.rb attributes definitions files providers
recipes resources templates}

  rm_rf sandbox
  mkdir_p sandbox
  cp_r Dir.glob("{#{files.join(',')}}"), sandbox
  puts "\n\n"
end

```

This Rakefile will copy the contents of our cookbook to a temporary directory and run the foodcritic tests on the temporary directory.  Note the `--epic-fail` tag is used to fail the build (return a non-zero exit code) on `any` rule that does not pass.

That's it!  When you push your commit to github, you should see Travis CI pick-up the changes, run your build, and report on status.

### Share Your Build Status

One final step that you may consider is adding a build status indicator to your README.  This simple line in your README will let others know what the current build status is for your cookbook.

``` sh

[![Build Status](https://secure.travis-ci.org/[YOUR_GITHUB_USERNAME]/[YOUR_PROJECT_NAME].png)](http://travis-ci.org/[YOUR_GITHUB_USERNAME]/[YOUR_PROJECT_NAME])

```

### Thanks & Additional Resources

A big "Thank You!" shout-out to [Fletcher Nichol](https://twitter.com/fnichol) and [Eric G. Wolfe](https://twitter.com/atomic_penguin) from whom I 'borrowed' the `Rakefile` and `.travis.yml` used in this post.

More information on Foodcritic and Travis CI can be found here:

* [Foodcritic](http://acrmp.github.com/foodcritic/)
* [Travis CI: Getting started guide](http://about.travis-ci.org/docs/user/getting-started/)
* [Travis CI: Status Images](http://about.travis-ci.org/docs/user/status-images/)

---
<sub>Reposted from [Nathen Harvey's blog](http://nathenharvey.com/blog/2012/05/29/mvt-foodcritic-and-travis-ci/)<sub>
 
