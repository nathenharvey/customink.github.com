---
layout: post
title: "Welcome, Dan Hensgen"
date: 2012-06-27 16:00
comments: true
author: nathan-hessler
categories:
  - team
published: true
---


Let me start off by apologizing to Dan for such a belated welcome post. It is belated for good reason though. Normally we mark our new engineer's first deploy. Today does not mark Dan's first deploy to production. But today's deploy is the culmination of many deploys. And, it has been the one I've been waiting for.

![Long Day at Work](/images/dan_hensgen_the_new_hotness.jpg)

<!-- more -->

Dan actually started in the middle of May, and we immediately began working on a maintenance project. we would be replacing a reliable, but old application running ruby 1.8.3 and rails 1.2.6 to use ruby 1.9.x and rails 3.2.x. we'll call the old app OldNotBusted and the new app TheNewHottness. along with this update we would decouple TheNewHottness from the couplings of OldNotBusted to allow for new needs within the company. This update and decoupling also meant updating code in other apps that communicate with OldNotBusted to communicate with TheNewHotness.

Remember how I said this is not Dan's first deploy. Well, first off TheNewHottness is new so it had nothing talking to it and could go to production very easily. And, in the other applications we have been doing what I will call "preploys". We have, at proper milestones, merged our code in the other applications and tested it, staged it, and deployed it. We did not remove any of the old code and the code we wrote was not being exercised in the other apps. But, the "preploys" allowed us to test the code in those environments and kick off calls to TheNewHottness manually using rails console and such. Also, we could verify our changes were not breaking other parts of those apps in unexpected ways. And Finally, "preploys" allowed us to make the final deploy very undramatic. It was a flip of the switch so to speak and now TheNewHotness is running and accepting calls that OldNotBusted used to take. The next deploy will by my favorite. Legacy code removal.

I'll finish by saying it has been a joy to work with Dan on TheNewHotness. And, I'm looking forward to working with him on future projects. Glad to have you aboard at CustomInk.

