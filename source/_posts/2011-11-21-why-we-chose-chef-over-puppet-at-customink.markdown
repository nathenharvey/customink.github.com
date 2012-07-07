---
layout: post
title: "Why we chose Chef over Puppet at CustomInk"
date: 2011-11-21 03:29
published: true
comments: true
author: nathen-harvey
categories:
  - chef
  - devops
  - heroku
  - puppet
  - sysadmin
  - web operations
---
Not unlike most technology choices, the choice of which configuration management tool to use for managing your infrastructure as code is sure to spark debate among opinionated technologists. There are certainly a number of choices available all of which have their own strengths and weaknesses. There are a number of things to consider as you select a tool.

Before we get into any of the specifics, I want to make it clear that the "right" answer to this question is a simple, but emphatic "yes!" Yes, you should be using a tool that allows you to manage your infrastructure as code. That tool should NOT be a server.txt file that you keep on the machine that documents the installation, set-up, and configuration changes you've made. Moving that text to somewhere other than the local server is a step in the right direction but isn't really the answer. Moving the server.txt file to your corporate wiki is going to suck just as bad.

I think Mark Imbriaco summed it up quite nicely in 140 characters or less:

{% blockquote @markimbriaco https://twitter.com/markimbriaco/statuses/89180299824599041 %}
Pro-tip: Nobody gives a shit about your opinion of Chef vs. Puppet. Seriously. Just fucking stop it already. #usewhatworksforyou
{% endblockquote %}

As you consider which tool is right for you, you'll need to consider a number of questions. I think of these as the WIIFs, or "what's in it for..." questions:
<!--more-->
### WIIFM - What's in it for me
You're going to want a tool that you're happy working with. You're going to make an investment in this tool. You'll need to learn to be proficient with the tool, master it, and use it in your everyday workflow. Pick something that you'll be happy working with for some time.

### WIIFC - What's in it for my customers
It's highly unlikely that your customers know or care anything about how your infrastructure was built, provisioned, and managed. Why should they have any say about which tool you pick? Your customers are keenly interested in the services or products you offer. They also care about things like performance, availability, and how quickly you recover from an issue or outage. If they don't care about these things, they'll likely not be your customers for long. As you grow your business, you'll want to have more time for delivering value to customers. Spend less time building, provisioning, upgrading, and repairing your infrastructure.

### WIIFB - What's in it for my business
You may be the only one who has to build and manage the infrastructure in your company but it's likely you'll eventually move beyond a technology team of one. As your technology team grows, you'll want to include everyone in the process of managing your infrastructure. This includes the people you might not think of as typically having a say in the infrastructure: developers, quality assurance engineers, etc. You do not want to be the only person in your company who knows how to manage the infrastructure and use the tools you've selected. Sure, it gives you a false sense of job security and feeds into your hero-complex but you need to be able to pass the on-call baton to someone else. Cost may also be a factor to consider when selecting a solution although it's likely it's more of a data-point than selection criteria, given the solutions that are on the market.

I cannot tell you which tool is right for you. There are many factors including the ones I've listed above. I have some experience with both Puppet and Chef. At [CustomInk](http://www.customink.com), we decided to switch to Chef after using Puppet for about two years.

## Why did we switch?
### We're a Rails shop
CustomInk is a Ruby on Rails shop and has been for many years. Being a Rails shop helped push us towards Chef in two ways. As a Rails shop, we suffer a bit from from the stereotypical "newer and shinier is better" syndrome that many people feel ails the Rails community. As a Rails shop, the domain-specific language (DSL) of Chef is a more comfortable way for us to work. Everyone on the technology team can easily understand the code.

### We started with Puppet
We started with Puppet so, naturally, that's the one we switched from. Puppet was, and actually still is, working well for us. However, we found that working in Puppet was going a bit slower than we'd like. Also, as we started learning more about Chef we started to see how we'd be able to quickly benefit from some of the features it offers. To be fair, we were comparing the Puppet we were using (0.24.x) to the latest (at the time) version of Chef (0.9.x). There may well have been ways to do the things we wanted with Puppet but we weren't. Chef was intriguing and it looked like we'd be able to get more from it. Instead of working to refactor our Puppet and get smarter with how to use it, we went with Chef.

### Search
Chef's ability to search our environment and use that information at run time is very appealing. The ability for us to define a database.yml template that can have the "host" value populated at runtime based on which host is currently the primary database server is great. Using search in our capistrano recipes to determine where the code should be deployed is a huge win.

### Knife
Knife is Chef's powerful command line interface. There are many things you can do with knife, most of which fall outside of the scope of this article. Knife allows you to interact with your entire infrastructure and Chef code base. Use knife to bootstrap a server, build the scaffolding for a new cookbook, or apply a role to a set of nodes in your environment. You can use knife ssh to execute commands on any number of nodes in your environment. knife ssh + search is a very powerful combination. "Run this command on all nodes with role X."

### Dependency Management
We found that defining dependencies in Puppet was overly verbose and cumbersome. With Chef, order matters and we could rest assured that dependencies would be met if we specified them in the proper order.

### Strong Community
OpsCode has done a great job of keeping up a strong community. The community.opscode.com site, where hundreds of cookbooks are shared, is a great way to get started. OpsCode has also hosted numerous webinars, publishes all of their training material, and makes it very easy to contribute patches. Frankly, I don't have any experience with this in the Puppet world. However, my lack of experience with this in the Puppet world is likely attributed simply to the way my development habits have changed over time. At CustomInk, we've been able to submit patches to chef, a number of cookbooks, and have also published some of our own cookbooks.

### Developer Happiness
As I mentioned previously, the DSL with Chef is much more comfortable than that of Puppet. The mental model and workflow suit us. I find that the time I spend working in Chef is when I feel most productive and happy.

I often wonder if the reason Chef is the right tool is because it's the second one we've used. Coming to infrastructure as code includes a learning curve. I feel that we're better Chef developers because we learned from our experience with Puppet. Some may even agree that Chef's a better tool because the developers of Chef learned from their experience with Puppet.

### A note for projects that are just getting started
If your project is just getting started, the best choice for you is probably not to use any of the configuration management tools that allow you to manage your infrastructure as code. You should stay focused exclusively on delivering value to your customers. It's likely that the best solution for you is [Heroku](http://www.heroku.com/). Sure, Heroku puts some constraints on how you build your app, but they're a good way for you to think creatively. You can, and should, delay your choice of tools until you're ready to spin up your first server.

## TL;DR

* If the question is "Chef or Puppet?", the answer is "Yes." You need to manage you infrastructure as code
* Search, knife, dependency management, community, and developer happiness were the key reasons we switched
* Chef is the right tool for us and it might be the right one for you
* If you're new to the idea of "Infrastructure as Code", understand that there's a learning curve but your efforts will be rewarded

Did you have to make a similar choice? What were some of the deciding factors? Which tool or framework did you end up with?

---
<sub>Reposted from [Nathen Harvey's blog](http://nathenharvey.com/blog/2011/11/21/why-we-chose-chef-over-puppet-at-customink/).</sub>

