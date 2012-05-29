---
layout: post
title: "Provision your laptop with Chef: Part 1"
date: 2012-05-28 8:32pm
comments: true
categories:
  - chef
  - utilities
  - devops
  - webops
  - tutorial
author: seth-vargo
published: true
---

If you have ever tried to follow the [Opscode Getting Started Guide for Chef](http://wiki.opscode.com/display/chef/Getting+Started), you'll quickly be overtaken by Chef jargon, confusing instructions, many assumptions, and no clear direction. Even the most experienced developers had a difficult time following the Opscode Wiki. While it serves as a great reference resource, you pretty much have to know Chef before it is of any use.

In Part 1 of this 2-part series, we will walk through setting up the Chef environment on your local machine. In Part 2, we will demonstrate how to provision your personal laptop using Chef.

What is Chef?
------------
One of the best things about Chef is that it is idempotent. Unlike a simple bash script or complex startup script, Chef can be applied to the same machine hundreds of times. Only the "changes" will be executed on each of the clients. This makes deploying and updating thousands of machines as easy as baking a cake!

Understanding Chef & Jargon
---------------------------
One of the most interesting and confusing parts about Chef is its lingo. Everything relates to **food**. Be sure to grab a snack before diving in any further.

### Architecture
One of the most dangerous practices when using Chef is applying prior knowledge. In Chef, everything is named relative to Chef. This means, even though your are provisioning a production *server*, it's still a Chef *client*. This can be very confusing at first, but once you start thinking in terms of Chef things will begin to make sense.

 - `(Chef) Server` [aka hosted chef] - refers a machines that stores cookbooks, roles, and other Chef configurations
 - `(Chef) Client` [aka node] - refers to a machine that connects to and is managed by a Chef Server
 - `(Chef) Workstation` - refers to a machine where recipes are developed, altered, deleted, and more

This is the most simplistic picture. It's possible for a single machine to exist in all three of these roles simultaneously! There are subsets of Chef, such as [Chef-Solo](http://wiki.opscode.com/display/chef/Chef+Solo), but those will not be discussed here.

A typical scenario begins at the workstation. A developer creates a cookbook, role, or other artifact on the local machine. When finished, that artifact is uploaded to the Chef Server. The Chef Clients receive these (new) instructions and execute them locally.

![Chef Architecture Graph](/images/chef-architecture.png)

### Jargon
We've also been throwing around some other terms that we should take a second to define:

 - `cookbook` - collection of recipe, resources, attributes, templates, metadata, and other files
 - `recipe` - a set of instructions written in a Ruby DSL that tells a node what commands to execute
 - `metadata` - additional information, such as dependencies, for a given cookbook
 - `resource` - cross-platform abstraction of the "thing" you're configuring such as a package or a user
 - `provider` - a platform-specific implementation of a resource
 - `data bag` - JSON key-value store for storing data, attributes, and more
 - `environment` - provide a mechanism for managing different deployment locations such as production, staging, development, and testing
 - `template` - a file (like a config file) to be "rendered" on the server

 There are also a few tools:

  - `knife` - a command line tool for managing chef and chef recipes
  - `shef` - chef console. this is the equivalent of `rails console` for chef

Before continuing, please make sure you understand that these are very over-simplified definitions and abstractions. Chef is much more powerful that these simplistic definitions allow.

Set up your Opscode Account
---------------------------
For the purposes of this tutorial, we will use Chef Hosted by Opscode (the creators of Chef). The service provides 5 free nodes for use. Go ahead and create your free account by heading over to the [Hosted Chef Signup page](https://community.opscode.com/users/new). You should see something like this:

![Hosted Chef Signup page](/images/hosted-chef-signup.png)

You'll need to confirm your email, but once that's done, head on over to the [Opscode Management Console](https://manage.opscode.com/organizations). It really doesn't matter what you call your organization - I used the same as my username. You should see a screen like this:

![Hosted Chef Management Console](/images/hosted-chef-management-console.png)

You probably only have one organization, but the image should give you a good idea. First thing you'll want to do is **Regenerate validation key** and **Generate knife config**. We will talk about this more in detail, but save these files in a handy place for later. These files should always be kept securely.

Also, if you didn't get your private key when registering, you should do that now (for some reason, Opscode does not always stream the key). Go to the [Opscode Community Site](http://community.opscode.com/) and login (if you aren't already). Click on your profile and then choose **get private key**.

![Chef Community User](/images/chef-community-user.png)

At this point you should have the following files:

    [your_organization_name]-validator.pem
    [your_username].pem
    knife.rb

For example, mine would be:

    ci-validator.pem
    sethvargo.pem
    knife.rb

Sometimes Opscode doesn't stream the correct files, so you may need to do some renaming:

    _knife_config   #=> knife.rb
    _regenerate_key #=> [your_organization_name]-validator.pem

Install Ruby, Chef, and Git
---------------------------
Chef is written in Ruby. Therefore, you must have Ruby installed. Installing Ruby is beyond the scope of this topic, but here are some quick resources:

 - [Ruby Installer for Windows](http://rubyinstaller.org/) (also install the [Development Kit](https://github.com/oneclick/rubyinstaller/wiki/development-kit))
 - [Installing Ruby with rvm](http://rvm.io/)
 - [Installing Ruby with rbenv](https://github.com/sstephenson/rbenv)

You should also install `git`. Check out the [Github tutorial for installing git](http://help.github.com/set-up-git-redirect).

Once you have Ruby and Git installed, you'll need to install chef:

    gem install chef

This command will install the Chef gem as well as some other dependencies.

A this point, you should have a novice understand of Chef and Chef Jargon, have an account on Opscode Hosted Chef, and have a working version of Ruby, Git, and the Chef gem installed. The rest of this guide will assume you have completed all those steps correctly.

Setup your Workstation
----------------------
Find a working directory on your local machine where you plan to store your Chef cookbooks. For this tutorial, we will use `~/Development`. You'll need a skeleton chef repository. You could make your own or clone the Opscode one:

    git clone git@github.com:opscode/chef-repo.git

Take a few minutes to poke around the repository. You definitely won't understand everything, but look at a few READMEs (in subdirectories).

Inside the `chef-repo` directory (`~/Development/chef-repo`), we need to create a hidden folder named `.chef`:

    cd ~/Development/chef-repo
    mkdir .chef

This is where you should put the files we downloaded earlier:

    mv [location]/[your_username].pem ~/Development/chef-repo/.chef/
    mv [location]/[your_organization_name].pem ~/Development/chef-repo/.chef/
    mv [location]/knife.rb ~/Development/chef-repo/.chef/

For my case, it would be:

    mv ~/Desktop/sethvargo.pem ~/Development/chef-repo/.chef/
    mv ~/Desktop/ci-validator.pem ~/Development/chef-repo/.chef/
    mv ~/Desktop/knife.rp ~/Development/chef-repo/.chef/

To confirm everything is working, try and list all the clients (it should be empty):

    knife client list

If this command succeeds without error, everything is set up correctly!

This concludes setting up your workstation. Now we are ready to create our first cookbook.

Create your first cookbook
--------------------------
Wash your hands, put on your Chef's Hat, and get ready to bake! We are going to create your very first cookbook. The cookbook will be very simple and is more for demonstrating working with `knife` and `chef-client`.

Our cookbook will be named "hello world". Let's start by using `knife` to create our cookbook skeleton:

    cd ~/Development/chef-repo
    knife cookbook create hello_world

You should see the following output:

    ** Creating cookbook hello_world
    ** Creating README for cookbook: hello_world
    ** Creating metadata for cookbook: hello_world

Open up the project in your favorite text editor and look inside the `cookbooks` directory. You should see the following:

![Chef Cookbook Structure](/images/chef-cookbook-structure.png)

This cookbook will create a file `~/hello_world.txt` that says "Hello World!".

Let's first make the template. A template is like the "view" of MVC. It has access to instance variables and uses embedded ruby. Inside the `templates/default` directory, create a new file named `hello-world.txt.erb` and add some content:

```erb
<% # templates/default/hello-world.txt.erb %>
Hello World!

Chef Version: <%= node[:chef_packages][:chef][:version] %>
Platform: <%= node[:platform] %>
Version: <%= node[:platform_version] %>
```

You can see we are referencing a `node` variable. This refers to the current client that Chef is running on.

Now we need to create the actual recipe with instructions. You can think of the recipe as the "controller" of MVC. Open up the `recipes/default.rb` file and add the following:

```ruby
# recipes/default.rb
template "#{ENV['HOME']}/hello-world.txt" do
  source 'hello-world.txt.erb'
  mode '0644'
end
```

The `template` keyword is a resource like we defined before. It tells the recipe to render the given template source to the given file. We also specify the file permissions using `mode`.

That's it! Our cookbook is done and ready to be uploaded to Hosted Chef. We will use the `knife` command to do this:

    knife cookbook upload hello_world

You should see output like this:

    Uploading hello_world             [0.0.1]
    Uploaded 1 cookbook.

If you get a message like:

    WARNING: No knife configuration file found
    ERROR: Your private key could not be loaded from /etc/chef/client.pem
    Check your configuration file and ensure that your private key is readable

it means that your key is invalid. Regenerate your personal and/or organization keys and ensure everything is placed in the correct directories.

Our cookbook is now on Hosted Chef and ready to be distributed to our nodes. Chef doesn't automatically tell nodes to update. You can do this with a cron job, running `chef-client` as a service, or by running `chef-client` manually on any node.

For simplicity, let's just run this on our local workstation. Let's set up the local workstation as a client. Run the following command from the inside the repository:

    sudo knife configure client /etc/chef

If you're using `rvm`, use the `rvmsudo` command prefix:

    rvmsudo knife configure client /etc/chef

You should see output like:

    Creating client configuration
    Writing client.rb
    Writing validation.pem

That's it! Your workstation is now a client. Let's provision this server by running `chef-client`:

    sudo chef-client

Or with RVM:

    rvmsudo chef-client

You should see output like this:

    INFO: *** Chef 0.10.10 ***
    INFO: Client key /etc/chef/client.pem is not present - registering
    INFO: Run List is []
    INFO: Run List expands to []
    INFO: Starting Chef Run for NODE
    INFO: Running start handlers
    INFO: Start handlers complete.
    INFO: Loading cookbooks []
    WARN: Node NODE has an empty run list.
    INFO: Chef Run complete in 0.988165 seconds
    INFO: Running report handlers
    INFO: Report handlers complete

Take a look inside your `$HOME` directory and see if the `hello-word.txt` file was created...

It doesn't appear the file was created. We must have done something wrong! Actually, we did everything correctly. We just forgot one step - we never **told** our node to execute the recipe we just wrote. By default, Chef does not execute any of your recipes. You must explicitly require them. There are a variety of ways to do this. We will use a `run_list` here.

First, we need to figure out what our node is named. Run `knife node list` and you should now see two nodes:

    NODE
    [your_organization_name]-validator

Mine looks like:

    seth
    sethvargo-validator

We obviously want `NODE`. In my case, it's `seth`. Now we can edit the `run_list` for that node:

    knife node run_list add NODE hello_world

For me, that command would look like:

    knife node run_list add seth hello_world

You should see the following output:

    run_list:  [recipe[hello_world]]

Let's inspect this node using the `show` command:

    knife node show NODE

You should now see `hello_world` in the `run_list`. Run the `chef-client` command again (remember you might need to use `sudo`) and you should get output like the following:

    INFO: *** Chef 0.10.10 ***
    INFO: Run List is [recipe[hello_world]]
    INFO: Run List expands to [hello_world]
    INFO: Starting Chef Run for seth
    INFO: Running start handlers
    INFO: Start handlers complete.
    INFO: Loading cookbooks [hello_world]
    INFO: Storing updated cookbooks/hello_world/recipes/default.rb in the cache.
    INFO: Processing template[/Users/seth/hello-world.txt] action create (hello_world::default line 10)
    INFO: template[/Users/seth/hello-world.txt] mode changed to 644
    INFO: template[/Users/seth/hello-world.txt] updated content
    INFO: Chef Run complete in 1.188536 seconds
    INFO: Running report handlers
    INFO: Report handlers complete

Open up your `$HOME` directory and you should see a file named `hello-world.txt`. Look inside and you'll see the those node variables were translated into plain text. Awesome!

You can delete that file or leave it around as a reminder of how awesome Chef is. We have one last thing to do before we are done with this tutorial.

Let's remove that recipe from the `run_list`:

    knife node run_list remove NODE hello_world

Again, mine would be:

    knife node run_list remove seth hello_world

You should see output like the following:

    run_list:  [recipe[hello_world]]

Well, this concludes this (rather lengthy) tutorial on installing Chef, registering for Hosted Chef, creating your first cookbook, and provisioning your first machine.

Part 2 of this series will cover more recipes and full provisioning and customization of your personal laptop using Chef.
