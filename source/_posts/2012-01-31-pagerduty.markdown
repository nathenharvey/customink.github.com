---
layout: post
title: "PagerDuty, Nagios and Chef"
date: 2012-01-31 10:14
author: jake-vanderdray
comments: true
published: true
categories:
  - chef
  - nagios
  - pagerduty
  - web operations
---

## Three Things that Work Great Together

If you use Chef and Nagios, you already know what a great combination they make.  As you build new servers they automatically start getting monitored by Nagios.  Without you having to do anything they're grouped together based on role, so its easy to apply the same checks for all servers in a given role.  If you haven't tried Nagios built with the chef cookbook its easy to get started with this [guide](http://wiki.opscode.com/display/chef/Nagios+Quick+Start) from Opscode.

[PagerDuty](http://www.pagerduty.com/]) is a service that manages your on-call alerting and escalation policies.  Its hard to love a service that wakes you up in the middle of the night telling you about problems with your servers (my wife is really not a fan), but PagerDuty is helpful.  We generally set it to send an email about a problem first, then send an SMS text and finally to actually make a phone call if no one has responded.  It will go through a rotating list of people on call and accepts alerts from a number of monitoring services including Nagios and AlertSite.

<!-- more -->

Opscode recently accepted my addition of a PagerDuty recipe to the [Nagios cookbook](https://github.com/opscode/cookbooks/tree/master/nagios) which makes it incredibly easy to connect your Nagios instance to PagerDuty.  You just add a PagerDuty API key as an attribute, apply the pagerduty recipe to your nagios server (see their [guide](http://www.pagerduty.com/docs/guides/nagios-integration-guide) for instructions on getting your key) and you're good to go.

If you add the API key(s) to your Chef environments, you can tie each environment to a different escalation policy.  That way your staging environments just send email alerts while production actually texts and calls.

One of the big wins of using the Nagios plugins is that if a service recovers the PagerDuty incident gets resolved automatically so it doesn't continue to escalate the problem.  Also if you acknowledge a problem in Nagios the acknowledgment flows through to PagerDuty.
