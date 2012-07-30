---
layout: post
title: "Provision your laptop with Chef: Part 2"
date: 2012-07-28 8:32pm
comments: true
categories:
  - chef
  - utilities
  - devops
  - webops
  - tutorial
  - followup
author: seth-vargo
published: true
---

In [Provision your laptop with Chef: Part 1](/blog/2012/05/28/provision-your-laptop-with-chef-part-1/), I showed you how to setup your free Opscode Chef account and register your local development machine as a client. In Part 2, we will explore a few handy ways to manage and provision your new or existing laptop with Chef. Whether it's creating users, installing applications, or always having the perfect desktop background, Chef can make it happen!

Part 2 of this series assumes that you have successfully completed all of the steps in [Part 1](http://technology.customink.com/blog/2012/05/28/provision-your-laptop-with-chef-part-1/). If you have not yet completed Part I, please do so before continuing.

<!-- more -->

Overview
--------
In this blog post, we will be using an Ubuntu 12.04 Precise desktop environment. These steps may be tweaked to work on Windows or Mac (mainly just changing the file paths). You could even bake conditional logic into your cookbooks so that it will work on all your laptops, Mac, PC, Linux or otherwise!

My local chef repository lives at `~/Development/chef-repo`. I will use this path throughout this blog post. You do not need to have your chef-repo in the same place - just replace my path with yours where appropriate.

Starting Small
--------------
To get your feet wet, we will create a cookbook for a very simplistic task - setting the Desktop background.

Before beginning anything with Chef, you should check the [Chef Community Site](http://community.opscode.com) - someone may have already done the hard work for you! Even if you can't find exactly what you are looking for, explore a little. You may find small syntax shortcuts or new ways to solve problems from looking at various cookbooks.

I did a quick search on the Chef Community Site for "desktop" and "background" and it doesn't look like there's anything we can use there, so we'll have to build our own.

First, go into your local chef repository:

    $ cd ~/Development/chef-repo/

and make sure you have a clean working directory:

    $ git status
    # On branch master
    nothing to commit (working directory clean)

If you have unstaged commits, make sure to either stash or commit them before continuing.

Before we create our first cookbook, let's add a couple of things to our `knife.rb` file to make the process go a little more smoothly.  Open up your `knife.rb` file and add the following two lines to the end of the file replacing the sample values with your information.  (NOTE:  if you set-up your chef-repo like the [previous post]((/blog/2012/05/28/provision-your-laptop-with-chef-part-1/)), you'll find the file at `~/Development/chef-repo/.chef/knife.rb`.)

    cookbook_email      'email@example.com'
    cookbook_copyright  'Your Name'

With these new knife values, we're ready to create our first cookbook. This cookbook will manage our Desktop background. If we were creating a cookbook that installed a particular piece of software, like ImageMagick, convention would dictate that we name it after that software. However, in this case, we are free to choose any name we'd like. You could be boring and pick something like "desktop-background", but that's no fun. Let's name our cookbook "major-tom" (background-control => ground-control => major-tom for those following along at home). Use the `knife cookbook create` command to generate the cookbook skeleton:

    $ knife cookbook create major-tom

**Note:** You may need to prefix the command with `bundle exec`

You should see output like this:

    ** Creating cookbook major-tom
    ** Creating README for cookbook: major-tom
    ** Creating metadata for cookbook: major-tom

Take a moment to explore the new cookbook. You should have a directory structure like this:

    major-tom
    |_attributes
    |_definitions
    |_files
      |_default
    |_libraries
    |_providers
    |_recipes
      |_default.rb
    |_resources
    |_templates
      |_default
    |_metadata.rb
    |_README.md

The **first** thing you should do before continuing is to create a `CHANGELOG.md` file in the cookbook root. This will be unnecessary if [this pull request](https://github.com/opscode/chef/pull/345) is merged. Add an entry indicating our initial release:

```text
CHANGELOG
=========
### v0.0.1
- Initial release
```

Next, we need to fix up the `metadata.rb`. Update the description to include a bit more detail.  Everything else should be OK.

```ruby
maintainer 'Your Name'
maintainer_email 'email@example.com'
license 'All rights reserved'
description 'Installs/Configures desktop background images'
long_description IO.read(File.join(File.dirname(__FILE__), 'README.md'))
version '0.0.1'
```

In this scenario, let's imagine that we want to download our desktop image from the Internet. We will use the [Remote File](http://wiki.opscode.com/display/chef/Resources#Resources-RemoteFile) resource to fetch the remote image:

```ruby
# recipes/default.rb
remote_file "#{Chef::Config[:file_cache_path]}/background.png" do
  # ...
end
```

Notice how we **don't** hardcode the temporary path. You may be tempted to do something like:

```ruby
remote_file '/tmp/background.png' do
  # ...
end
```
but this is generally considered bad practice and will not pass [foodcritic](http://acrmp.github.com/foodcritic/#FC013) tests. Use Chef's `:file_cache_path` configuration option instead. This is especially important when developing a community cookbook.

Next we need to give the background image a `source` url. For copyright reasons, I'm going to use a [placehold.it](http://placehold.it) to generate our image. You could use any image you'd like, including one directly inside the cookbook (you would need to use the `cookbook_file` resource instead of `remote_file` in that case). I have a high resolution display, so I want a background image 1920x1080 (HD):

```ruby
remote_file "#{Chef::Config[:file_cache_path]}/background.png" do
  source 'http://placehold.it/1920x1080'
end
```
This declaration says:

> "Hey Chef, go out to [http://placehold.it/1920x1080](http://placehold.it/1920x1080) and download
> and save whatever you get to your file cache path. kthxbye."

If you're a super-nerd, this declaration says

```bash
(cd /tmp && wget -O background.png http://placehold.it/1920x1080)
```

Now that we have the remote file downloaded to the system, we need to programatically set it as the background. In your head, you may think:

> Okay, I need to right-click on that image and click on the `Set as Desktop Background` option...

Unfortunately, Chef doesn't work quite that way :). All the commands you run are executed in the context of a command line or terminal. We need to find a way to use the command line to set the Desktop background. Fortunately, most of the GUI commands on your machine actually use underling command line tools to execute. A quick Google search turned up [Setting Desktop Wallpaper via Terminal](http://askubuntu.com/questions/51564/setting-desktop-wallpaper-via-terminal) and [How do I change the desktop background from command line?](http://askubuntu.com/questions/50019/how-do-i-change-the-desktop-background-from-command-line).

It turns out we need to do something like this:

```bash
gconftool-2 -t string -s /desktop/gnome/background/picture_filename <path>
```

Because this is a bash command, we could use the `Bash` resource. However, I like the `execute` resource, because it is more Rubyesque in my opinion:

```ruby
execute 'set Desktop background' do
  command "gconftool-2 -t string -s /desktop/gnome/background/picture_filename #{Chef::Config[:file_cache_path]}/background.png"
  action :nothing
end
```

Notice how I set the `action :nothing`. This may seem strange. Don't we run this code to run? (Note, the default action is `:run`). Well, we only want to run this command once the `remote_file` has completed successfully. Otherwise, this command will fail. We need to modify our `remote_file` declaration to execute this command:

```ruby
remote_file "#{Chef::Config[:file_cache_path]}/background.png" do
  source 'http://placehold.it/1920x1080'
  notifies :run, 'execute[set Desktop background]', :immediately
end
```

This now says:

> "Hey Chef, go out to [http://placehold.it/1920x1080](http://placehold.it/1920x1080) and download
> and save whatever you get to your file cache path.
> Then tell `set Desktop background` to run immediately."

This ensures that:

1. The execute command won't be run before the remote file is downloaded.
2. The execute command only executes if the remote file was downloaded successfully.

Now we need to add this recipe to the node's `run_list`. I won't go into too much detail here. A quick solution is to do something like this:

```bash
knife node edit NODE
```

In the `run_list` part of the JSON file, add `recipe[major-tom]`:

```ruby
{
  "name": "NODE",
  "chef_environment": "_default",
  "run_list": [
    "recipe[major-tom]"
  ]
}
```

Save and close this file so that it is updated on the Chef Server.

Upload the cookbook (`knife cookbook upload major-tom`) and run `sudo chef-client`. You should see that the command completes successfully. However, your Desktop background may not change. Why? Well, by default, Chef runs as the root user. We just set the root user's background, but we wanted to set our own! After a bit of research, it turns out that we need to set the `user` attribute on the `execute` block to tell Chef which user to run the command as:

```ruby
execute 'set Desktop background' do
  command "gconftool-2 -t string -s /desktop/gnome/background/picture_filename #{Chef::Config[:file_cache_path]}/background.png"
  user 'svargo' # replace with your user id
  action :nothing
end
```

But what if you wanted to set the Desktop background for **all** the users? Maybe you operate in an Enterprise that requires the company logo be on the Desktop background. Or maybe you just have OCD and like everything to be the same. Either way, there are a few possible solutions for accomplishing this.  Let's look at one way to do this.

#### Using `node['etc']['password']`
You can iterate over all the local accounts on a given machine using the `node['etc']['passwd']` hash. It looks like this:

```ruby
node['etc']['passwd'].each do |user, data|
  execute 'set Desktop background' do
    command "gconftool-2 -t string -s /desktop/gnome/background/picture_filename #{Chef::Config[:file_cache_path]}/background.png"
    user user
    action :nothing
    only_if { data['uid'].to_i > 1000 }
  end
end
```

Here, we iterate over each user and execute the command once for each user. Notice the `only_if` block as well. This tells Chef to only run for non-system accounts (> 1000).

Upload the cookbook and run `sudo chef-client` again and you should see you Desktop background change. Depending on your machine, you may need to logout and login for the changes to take effect.

For those of you who cheated and read ahead, here's the full recipe:

```ruby
#
# Cookbook Name:: major-tom
# Recipe:: default
#
# Copyright 2012, Your Name
#
# All rights reserved - Do Not Redistribute
#

# Download the remote file
remote_file "#{Chef::Config[:file_cache_path]}/background.png" do
  source 'http://placehold.it/1920x1080'
  notifies :run, 'execute[set Desktop background]', :immediately
end

# Set the Desktop background for each user
node['etc']['passwd'].each do |user, data|
  execute 'set Desktop background' do
    command "gconftool-2 -t string -s /desktop/gnome/background/picture_filename #{Chef::Config[:file_cache_path]}/background.png"
    user user
    action :nothing
    only_if { data['uid'].to_i > 1000 }
  end
end
```

#### Wrap Up
There's definitely room for improvement on this cookbook. We could create attributes and allow the end user to customize it further. However, for the purpose of this tutorial, we are done working with this cookbook.

Diving Head First
-----------------
In this next step, I'm going to switch over to a Mac because I personally use a Mac. Before beginning, let's take a step back and ask ourselves:

> "What should I configure?"

This question is has potentially endless answers and all depends on how much time you want to spend capturing your current configuration in Chef. For this tutorial, I'm going to do the following:

1. Install the following packages:
    - `apple-gcc42`
    - `aspell`
    - `bash-completion`
    - `elasticsearch`
    - `erlang`
    - `ghostscript`
    - `git`
    - `imagemagick`
    - `jasper`
    - `mongodb`
    - `mysql`
    - `node`
    - `postgresql`
    - `qt`
    - `rabbitmq`
    - `readline`
    - `redis`
    - `solr`
    - `wget`
2. Clone a bunch of git repositories
3. Install my dotfiles

This is a very basic start and should cover many helpful topics for when you begin provisioning your own laptop.

Just as important as before, the cookbook name should be descriptive and fun. I'm going to call this cookbook `sethinator`, because my name is Seth, and I'm "Sethinating" my laptop.

    $ knife cookbook create sethinator

Be sure to change the `metadata.rb`, add a `CHANGELOG.md`, and everything else we did in the "Getting your Feet Wet".

### Installing packages
As you already know, there is no "built-in" package manager for Mac. Instead, there are two popular alternatives - homebrew and MacPorts. I will be using homebrew in these examples.

First, we need to tell Chef to use homebrew as the package manager:

    $ knife cookbook site install homebrew

This will download the homebrew cookbook from the [community site](http://community.opscode.com/cookbooks/homebrew).

We should also list this cookbook as a [dependency](http://wiki.opscode.com/display/chef/Cookbooks#Cookbooks-CookbookDependencies). Open up the `metadata.rb` and add the following:

```ruby
# metadata.rb
depends 'homebrew'
```

This way, if we choose to publish the cookbook to the community site, the homebrew cookbook will automatically be downloaded and installed with this cookbook.

To keep things organized, I'm going to create a separate recipe for each of the tasks above. Create a new recipe named `packages.rb` and add the following content:

```ruby
#
# Cookbook Name:: sethinator
# Recipe:: packages
#
# Copyright 2012, Seth Vargo
#
# All rights reserved - Do Not Redistribute
#
# Configures and installs the following packages using homebrew:
#   - apple-gcc42
#   - aspell
#   - bash-completion
#   - elasticsearch
#   - erlang
#   - ghostscript
#   - git
#   - imagemagick
#   - jasper
#   - mongodb
#   - mysql
#   - node
#   - postgresql
#   - qt
#   - rabbitmq
#   - readline
#   - redis
#   - solr
#   - wget
#

# Include homebrew as the default package manager.
# (default is MacPorts)
include_recipe 'homebrew'

# Install each of the packages using the `package` resource
%w(apple-gcc42 aspell bash-completion elasticsearch erlang ghostscript git imagemagick jasper mongodb mysql node postgresql qt rabbitmq readline redis solr wget).each do |package|
  package package
end
```

**That's it!** See how simple that was?

### Cloning git repositories
I have a very particular setup for my development workflow. Additionally, I regularly use a bunch of repositories. I could manually clone them, but why would I do that when I could manage them with Chef?

Create another cookbook named 'git.rb' in the recipes directory and add the following:

```ruby
#
# Cookbook Name:: sethinator
# Recipe:: git
#
# Copyright 2012, Seth Vargo
#
# All rights reserved - Do Not Redistribute
#
# Creates the ~/Development directory and installs git repositories
# specified as attributes on the node.
#

node['etc']['passwd'].each do |user, userdata|
  # Create the ~/Development directory
  directory "#{userdata['dir']}/Development" do
    owner user
    group userdata['gid']
    mode '0755'
    not_if { userdata['dir'].nil? || userdata['dir'] == '/var/empty' }
  end

  # Clone each git repository from the node's attributes
  #
  # We are using the `checkout` action because it will only checkout the
  # repository if it is not already there. We don't want a Chef run overwriting
  # local changes, but we do want an initial clone, so this is the best option.
  node['sethinator']['git'].each do |repository|
    git "#{userdata['dir']}/#{repository['name']}" do
      repository repository['url']
      reference repository['reference'] || 'master'
      revision repository['revision'] || 'HEAD'
      user user
      group userdata['gid']
      mode '0755'
      action :checkout
    end
  end
end
```

Slightly more complex than the last example, but still just a few lines of code. Notice that we haven't defined the git repositories inside the recipe. Instead they are attributes on the node itself. Create the default attributes file in `attributes/default.rb` and this is where I'm going to add all my repositories like so:

```ruby
# attributes/default.rb
default['sethinator']['git'] = [
  {
    :name => 'chef-hosted',
    :url => 'git@github.com:customink/...'
  },
  {
    :name => 'autotomy',
    :url => 'git@github.com:customink/...'
  },
  {
    :name => 'knife-spork',
    :url => 'git@github.com:jonlives/knife-spork'
  }
]
```

I've suppressed the URLs to our private repositories, but you get the general idea. Notice there are additional options such as `reference` and `revision` that I have left out because I'm okay with their default values. If you use another CVS such as SVN, you may want to support additional attributes and tweak accordingly.

### Installing dotfiles
Since I spend 90% of my time working in Terminal, I have a small [collection of dotfiles](https://github.com/sethvargo/dotfiles) that I've accumulated over the years. Everything from my bash prompt to RVM setup.

First, create a new cookbook named `dotfiles.rb` and add the following:

```ruby
#
# Cookbook Name:: sethinator
# Recipe:: dotfiles
#
# Copyright 2012, Seth Vargo
#
# All rights reserved - Do Not Redistribute
#

# Clone the remote dotfiles
#
# Again, only use :checkout because we don't want to clone on future attempts.
# We also don't plan on working with this repository, so we don't need the entire
# git history. Setting the depth to `1` makes a shallow clone. We use the git://
# url here because this computer (assuming it's new) does not have access to
# clone using its public key yet.
#
# Lastly, we only want to clone these if we haven't already installed our dotfiles.
# There are a few ways to do this, such as checking for the existence of a file,
# creating a file, checking the contents of a file, etc. Here we are just going to check
# and see if the `.gitconfig` file exists. That isn't created automatically with a new
# user, so it's a reasonable file to choose.
git "#{Chef::Config[:file_cache_path]}/dotfiles" do
  source 'git://github.com/sethvargo/dotfiles.git'
  user 'svargo'
  gid 'wheel'
  mode '0755'
  action :checkout
  depth 1
  not_if { ::File.exists?('/Users/svargo/.gitconfig') }
  notifies :run, 'execute[install dotfiles]'
end

# Run the dotfile rake task to install the files
gem 'rake'
execute 'install dotfiles' do
  cwd "#{Chef::Config[:file_cache_path]}/dotfiles"
  command 'rake install'
  action :nothing
end
```

That rake task will copy all the dotfiles into their proper locations. That's it!

### Put it all together
Open up the default recipe (`recipes/default.rb`) and add the following:

```ruby
#
# Cookbook Name:: sethinator
# Recipe:: default
#
# Copyright 2012, Seth Vargo
#
# All rights reserved - Do Not Redistribute
#

include_recipe 'sethinator::packages'
include_recipe 'sethinator::git'
include_recipe 'sethinator::dotfiles'
```

Now add `sethinator` (or whatever you called your cookbook) to your node's `run_list` and run `sudo chef-client`. You should have your bash prompt, configuration, aliases, and more right at your fingertips!

If you're looking for a more complete and customizable solution that you don't need to write from scratch, head on over to [Joshua Timberman's blog post](http://jtimberman.housepub.org/blog/2012/07/29/os-x-workstation-management-with-chef/). He goes into much more detail about MacOSX provisioning.

Caveats
-------
### Croning Chef
At [CustomInk](http://www.customink.com), we create a cron task to run Chef at a regular interval (on a randomly generated seed). This is great because we need only update a cookbook and all our servers will receive the change without having to touch them. However, when provisioning your local laptop, I do **not** recommend setting up a cron task for Chef. Because you are using the same machine for both writing Chef stuff and running Chef stuff, you could potentially brick your computer if you aren't careful. Because of this, I recommend running `chef-client` manually on your local machine.

### Development
Along similar lines as Croning Chef, I do not recommend that you "test" cookbooks on your local machine. Instead, replicate your current environment in a Virtual Machine. There are great tools out there like [Oracle's VirtualBox](https://www.virtualbox.org/) + [Vagrant](http://vagrantup.com). Run your cookbooks against a machine that you can easily snapshot and restore should something go awry, before pushing to "production" (your local machine).

### Workflow
If this is your first time using Chef, the workflow will feel awkward. There's a major disconnect between what is on your local box, what is on your CVS, and what is on the Chef Server (Opscode). Even if you are using semantic versioning, you may have different cookbook versions in different environments. First of all, don't worry. Chef is a big piece of software, and you can't expect to master it in a weekend. Secondly, there are a few tools that help you manage your workflow.

##### [Opscode Workflow Wiki](http://wiki.opscode.com/display/chef/Cookbook+Workflows)
The official Opscode wiki detailing various workflow implementations.
##### [knife-spork](https://github.com/jonlives/knife-spork)
Created by [Jon Cowie](https://github.com/jonlives) over at [Etsy](http://www.etsy.com/), KnifeSpork makes bumping, uploading, and sharing cookbooks a breeze, especially in collaborative environments.
##### [Librarian](https://github.com/applicationsonline/librarian)
Librarian is a tool for managing cookbooks. If you're familiar with Ruby, it's the equivalent of a `Gemfile`.

### Testing
I will not go into cookbook testing extensively here, but it's very important that your cookbooks follow some standard. At the time of this writing, there are a few solutions for testing cookbooks - [chefspec](https://github.com/acrmp/chefspec), [cucumber-chef](http://www.cucumber-chef.org/), [minitest-chef-handler](https://github.com/calavera/minitest-chef-handler), and [rspec-chef](https://github.com/calavera/rspec-chef) – and they each have their own distinct advantages. At the very least, you should run `knife cookbook test` and `foodcritic` against all your cookbooks. Nathen Harvey covered this in his [MVT: knife test and TravisCI](http://technology.customink.com/blog/2012/07/06/mvt-knife-test-and-travisci/) blog post.

When running foodcritic, I recommend adding both [CustomInk foodcritic rules](https://github.com/customink-webops/foodcritic-rules) and [Etsy foodcritic rules](https://github.com/etsy/foodcritic-rules). Clone the repositories (or use submodules) into a `foodcritic` directory in the root of your chef-repo:

```text
chef-repo
  |_cookbooks
  |_environments
  ...
  |_foodcritic
    |_customink
    |_etsy
```
Then you can run `foodcritic` like so:

```bash
bundle exec foodcritic -I foodcritic/* cookbooks/COOKBOOK
```

For more information on the rules provided, see the individual repos.
