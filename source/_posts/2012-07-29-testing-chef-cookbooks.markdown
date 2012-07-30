---
layout: post
title: Testing Chef Cookbooks
date: 2012-08-01
comments: true
categories:
  - chef
  - testing
  - devops
  - webops
author: seth-vargo
published: false
---

Throughout my internship at CustomInk, I've put a significant focus on Chef cookbook testing. At the time of this writing, there are a few solutions for testing cookbooks - [ChefSpec](https://github.com/acrmp/chefspec), [cucumber-chef](http://www.cucumber-chef.org/), [minitest-chef-handler](https://github.com/calavera/minitest-chef-handler), and [rspec-chef](https://github.com/calavera/rspec-chef) – and they each have their own distinct advantages. At the very least, you should run `knife cookbook test` and `foodcritic` against all your cookbooks. Nathen Harvey covered this in his [MVT: knife test and TravisCI](http://technology.customink.com/blog/2012/07/06/mvt-knife-test-and-travisci/) blog post.

At CustomInk, we test using [ChefSpec](https://github.com/acrmp/chefspec). Additionally, we use some home-grown gems such as [fauxhai](https://github.com/customink/fauxhai) and [Strainer](https://github.com/customink/strainer) to make testing easier.

<!-- more -->

Foodcritic
----------
Foodcritic is a linting tool for your cookbooks. Although technically not a "test", linting tools are frequently grouped with testing. Foodcritic is like jslint for cookbooks. **At the bare minimum, you should run `foodcritic` against all your cookbooks.**

As mentioned in [Nathen Harvey's](/blog/our-team/nathen-harvey.html) [MVT: knife test and TravisCI](/blog/2012/07/06/mvt-knife-test-and-travisci/), foodcritic does not verify that you have proper Ruby code. It is only a linting tool.

When running foodcritic, I recommend adding both [CustomInk foodcritic rules](https://github.com/customink-webops/foodcritic-rules) and [Etsy foodcritic rules](https://github.com/etsy/foodcritic-rules). Clone the repositories (or use submodules) into a `foodcritic` directory in the root of your chef-repo:

Strainer
--------
As you can see, that's a lot of typing just to test a single cookbook:

    bundle exec knife cookbook test COOKBOOK
    bundle exec foodcritic -I foodcritic/* cookbooks/COOKBOOK
    bundle exec spec

This is why I wrote [Strainer](https://github.com/customink/strainer). Strainer uses a `Colanderfile` at either the project-level or cookbook-level to run isolated tests on your cookbooks. The cookbook is actually copied to a temporary location and then the tests are run against it. To get started, create a Colander file in your editor and enter the following:

```bash
knife test: bundle exec knife cookbook test $COOKBOOK
foodcritic: bundle exec foodcritic -I foodcritic/* cookbooks/$COOKBOOK
chefspec: bundle exec spec
```

Notice

If you've used [foreman](https://github.com/ddollar/foreman), you should be familiar with this syntax.

Next, we want to `strain` out t

I recommend also checking out [Strainer on github](https://github.com/customink/strainer) for the most up-to-date documentation.
