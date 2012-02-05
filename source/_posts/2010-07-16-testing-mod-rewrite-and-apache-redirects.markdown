---
layout: post
title: "Testing mod_rewrite and apache redirects"
date: 2010-07-16 09:00
comments: true
author: Nathen Harvey
categories:
  - apache
  - devops
  - sysadmin
  - testing
  - web operations
  - Nathen Harvey
---
At [CustomInk](http://www.customink.com), we recently migrated from mongrel to Passenger for our Ruby on Rails website. This migration included a full rewrite of our apache configuration files.

With over 500 redirect and rewrite rules in place I needed a way to ensure my copy-n-paste skills were up to snuff and that we didn't loose any redirects along the way.

In my search for help, I found a [blog post by Patrick Reagan from Viget labs](http://www.viget.com/extend/test-drive-mod-rewrite-rules-with-testunit/) that described a method for writing tests that will verify all the rewrite rules and redirects. Patrick's ideas were packaged up into a gem and available on [github](http://github.com/eightbitraptor/http_redirect_test).

I can now write up tests like:

``` ruby
should_redirect "/cink/ideas/ideas.jsp", :to => "/inspiration/"
```

So now I can to some TDC (test-driven configuration) whenever I get a request for a new redirect.

What other methods have you used to test your rewrite rules?

---
<sub>Reposted from [Nathen Harvey's blog](http://nathenharvey.com/blog/2010/07/16/testing-mod-rewrite-and-apache-redirects/).</sub>

