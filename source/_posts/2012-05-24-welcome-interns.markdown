---
layout: post
title: "Welcome Interns!"
date: 2012-05-24 16:01
comments: true
categories: 
  - team
author: karle-durante
published: true

---
Last week the CustomInk Tech team welcomed two new interns, Nolan Carroll and Seth Vargo.  Nolan and Seth are joining us for the summer from Carnegie Mellon University where they are both majoring in Information Systems.  

Nolan and Seth wasted no time hopping on our [deploy train](/blog/2012/05/14/welcome-josh-born) last week, but they couldn't have done it without the help of our (semi) automated build process.  A while ago, Nathen Harvey talked about our [Green Screen](/blog/2012/01/02/green-screen/) build monitor.  While this serves as a great motivator for us to keep our builds passing, the ability for us to quickly create builds for any branch is what keeps us moving fast.

Using some home grown capistrano scripts, any developer can easily create and manage an automated Jenkins build, which will automatically be monitored by Green Screen.  A developer's typical workflow might look something like:

```ruby
  >> git branch new_feature
  >> git checkout new_feature
  .... code changes...
  >> git commit -m "added new feature"
  >> git push origin new_feature
```

Now that the a new feature branch is available in our remote repository, the developer can create and start their own Jenkins build:

```ruby
  >> cap jenkins:create
  "I've created rfe_new_feature."
  >> cap jenkins:build
  "I've found rfe_new_feature and it's building now."
```

Now a build is running on our Jenkins server.  More importantly, Jenkins will use it's Jedi powers to detect any changes to the remote branch and automatically kick off a new build.  When builds fail, the entire team is notified and Green Screen turns red.  We have a handful of other useful Jenkins tasks as well:

```ruby
  cap jenkins:build     # Kicks off a build for the current branch
  cap jenkins:console   # Shows the job's console
  cap jenkins:create    # Create a job for the current branch
  cap jenkins:delete    # Deletes a job with name = rfe_[current_branch]
  cap jenkins:list      # Lists all jobs in jenkins
  cap jenkins:status    # Gets the status of a job with name = rfe_[current_branch]
```

Continuous deployment can't happen without automation.  I'd like to give a big shout out to our DevOps team for driving us down the automation super highway.  I'd also like to extend a warm welcome to Nolan and Seth, we look forward to seeing their builds pass all summer long.