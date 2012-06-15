---
layout: post
title: "Welcome Jason Gilbertson!"
date: 2012-06-15 10:47
comments: true
categories: 
  - team
author: karle-durante
published: true
---

The CustomInk technology team would like welcome Jason Gilbertson.  Jason, a native of Iowa and a graduate of the Georgia Institute of Technology, has relocated himself to McLean, VA to join our team and we couldn't be more excited.

![Jason Gilbertson](/images/jason_gilbertson.jpg)

Like everyone on the CustomInk technology team, Jason was quick to create a [feature branch](/blog/2012/05/24/welcome-interns/) and then [deploy his first feature](/blog/2012/05/14/welcome-josh-born/) to production.  But there is another important aspect of our continuous deployment strategy that Jason participated in: feature verification.   

<!-- more -->

When engineers think they are done with the feature, they need to show it to the feature owners.  For internal features, the people who care are our internal business owners.  For customer facing features, our product managers take ownership.  The best way to show a feature to someone is to let them use it.  And the best way to let someone use a feature is to put it on an easily accessible "production like" environment (aka a 'Staging Environment').  

Thanks to the power of [Web Ops](/blog/2012/05/25/taming-the-kraken-how-operations-enables-developer-productivity/) we can easily [whip up a rails environment](/blog/2012/05/25/the-joy-of-cooking-whip-up-a-rails-environment-with-chef/) any time and any where we please.  With our Chef recipes, we can make use of Amazon Web Services to spin up temporary staging environments and give them friendly URL's that are easy for our feature owners to remember.  For instance, if feature branch is named "color_picker", the URL for the staging server would be "color-picker.staging.ci.com".

When a feature is "done", we stage it, drop a note in the ticket's comments indicating the staged URL, and then move on to the next ticket.  If there is feedback about the feature, we can easily make the changes and redeploy the branch to the staging environment.  Once the ticket is verified, we tear down the staging environment and deploy the feature to production.

Putting the new feature in a staging environment lets us do a few things.  First, it makes us more confident that our code can be deployed in a repeatable fashion (no surprises when we go to production).  Second, it let's more than one person use the feature at a time.  Lastly, it removes the necessity for people to "get together" to review changes.  

Since we work on tickets one at a time, we can follow a simple process to continuously deploy new features to our site and tools:  Branch, build, verify, deploy. 