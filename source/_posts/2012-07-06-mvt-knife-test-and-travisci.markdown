---
layout: post
title: "MVT: knife test and TravisCI"
date: 2012-07-06 22:52
author: nathen-harvey
comments: true
published: true
categories:
  - chef
  - devops
  - opschef
  - opscode
  - test driven development
  - testing
  - travisci
  - tutorial
  - web operations
  - webops
---
In my last post, [MVT: Foodcritic and Travis CI](http://technology.customink.com/blog/2012/06/04/mvt-foodcritic-and-travis-ci/) I described the process for having Travis CI look after your cookbooks and run Foodcritic, the cookbook lint tool, on your cookbook after each `git push`.  In this post, we'll iterate on the "Minimum Viable Test" idea by adding in support for knife's cookbook testing.

Wait, I'm already running foodcritic, do I really need to run `knife cookbook test`, too?

I'll use a very simple example to demonstrate that you do.

Let's create a very basic cookbook:

``` sh
knife cookbook create very_basic
** Creating cookbook very_basic
** Creating README for cookbook: very_basic
** Creating metadata for cookbook: very_basic
```
Next, we'll write a flawed recipe:

``` ruby cookbooks/very_basic/recipes/default.rb
package "flawed" do
  action :nothing
end
end
```

Now, run foodcritic on this cookbook:

``` sh
foodcritic cookbooks/very_basic
```

Foodcritic doesn't throw any errors or find any problem with the cookbook.

Let's try testing it with knife:

``` sh
knife cookbook test very_basic
checking very_basic
Running syntax check on very_basic
Validating ruby files
FATAL: Cookbook file recipes/default.rb has a ruby syntax error:
FATAL: /Users/nharvey/projects/chef-hosted/.chef/../cookbooks/very_basic/recipes/default.rb:22: syntax error, unexpected keyword_end, expecting $end
```

OK, it should now be obvious that `knife cookbook test` should be included as part of our MVT.

<!-- more -->

To get Travis CI running `knife cookbook test` for us, we'll need to add or update the following files:

* .travis.yml
* Rakefile
* test/.chef/knife.rb
* test/support/Gemfile

Of course, this assumes you've configured your cookbook as described in the [previous post](http://technology.customink.com/blog/2012/06/04/mvt-foodcritic-and-travis-ci/).  Let's start with the Rakefile.

``` ruby Rakefile
#!/usr/bin/env rake

desc "Runs knife cookbook test"
task :knife do
  Rake::Task[:prepare_sandbox].execute

  sh "bundle exec knife cookbook test cookbook -c test/.chef/knife.rb -o #{sandbox_path}/../"
end

task :prepare_sandbox do
  files = %w{*.md *.rb attributes definitions files libraries providers recipes resources templates}

  rm_rf sandbox_path
  mkdir_p sandbox_path
   cp_r Dir.glob("{#{files.join(',')}}"), sandbox_path
end

private
def sandbox_path
  File.join(File.dirname(__FILE__), %w(tmp cookbooks cookbook))
end
```

In the file snippet above, I've only included the parts that are relevant for getting knife working.  I'll include the full source of the Rakefile at the end of the article.

Next, let's add this rake task to our .travis.yml file.

``` ruby .travis.yml
language: ruby
gemfile:
   - test/support/Gemfile
rvm:
  - 1.9.2
  - 1.9.3
script:
  - bundle exec rake knife
```
To successfully run the knife command, Travis CI will need a very minimal Chef configuration.

``` ruby test/.chef/knife.rb
cache_type 'BasicFile'
cache_options(:path => "#{ENV['HOME']}/.chef/checksums")
```

And, of course, we'll need to add Chef to our Gemfile.  Be sure to specify a modern version as Travis CI will use 0.8.10 by default (at the time of this writing).

``` ruby test/support/Gemfile
source "https://rubygems.org"

gem 'rake'
gem 'chef', '~> 10.12.0'
```

That's it.  On your next `git push` Travis CI should run `knife cookbook test` on your cookbook.

## Running locally

To run the rake tasks locally, you'll need to tell bundler where the Gemfile is, or you'll need to move it to the root directory of your cookbook and update .travis.yml appropriately.  Use the following command to run your tests locally:

`BUNDLE_GEMFILE=test/support/Gemfile rake knife`
`BUNDLE_GEMFILE=test/support/Gemfile rake foodcritic`


## Full source code

You can checkout this [Github compare view](https://github.com/customink-webops/percona-install/compare/03b9446...d423b14) to see the changes made to the code from the [previous post](http://technology.customink.com/blog/2012/06/04/mvt-foodcritic-and-travis-ci/).


``` ruby test/.chef/knife.rb
cache_type 'BasicFile'
cache_options(:path => "#{ENV['HOME']}/.chef/checksums")
```

``` ruby .travis.yml
language: ruby
gemfile:
   - test/support/Gemfile
rvm:
  - 1.9.2
  - 1.9.3
script:
  - bundle exec rake knife
  - bundle exec rake foodcritic
```

The Rakefile was refactored a bit since the previous post:

``` ruby Rakefile
#!/usr/bin/env rake

task :default => 'foodcritic'

desc "Runs foodcritic linter"
task :foodcritic do
  Rake::Task[:prepare_sandbox].execute

  if Gem::Version.new("1.9.2") <= Gem::Version.new(RUBY_VERSION.dup)
    sh "foodcritic -f any #{sandbox_path}"
  else
    puts "WARN: foodcritic run is skipped as Ruby #{RUBY_VERSION} is < 1.9.2."
  end
end

desc "Runs knife cookbook test"
task :knife do
  Rake::Task[:prepare_sandbox].execute

  sh "bundle exec knife cookbook test cookbook -c test/.chef/knife.rb -o #{sandbox_path}/../"
end

task :prepare_sandbox do
  files = %w{*.md *.rb attributes definitions files libraries providers recipes resources templates}

  rm_rf sandbox_path
  mkdir_p sandbox_path
  cp_r Dir.glob("{#{files.join(',')}}"), sandbox_path
end

private
def sandbox_path
  File.join(File.dirname(__FILE__), %w(tmp cookbooks cookbook))
end
```

``` ruby test/support/Gemfile
source "https://rubygems.org"

gem 'rake'
gem 'foodcritic'
gem 'chef', '~> 10.12.0'
```

### Credit

A big "Thank You!" shout-out to [Seth Vargo](http://technology.customink.com/blog/our-team/seth-vargo.html) for writing most of the code used in this post!

---
<sub>Reposted from [Nathen Harvey's blog](http://nathenharvey.com/blog/2012/07/06/mvt-knife-test-and-travisci/)<sub>
