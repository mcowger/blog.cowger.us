---
title: 12 Factors in the Real World
date: '2016-08-19 15:54:00'
layout: post
---
For those of you that don't know, [Reddit](https://www.reddit.com/) is a pretty major site on the internet.  They bill themselves as the 'front page of the Internet', and they are[ #25 for traffic](http://www.alexa.com/siteinfo/reddit.com) globally.  They are based in San Francisco and have about 80 employees, the majority of which focus just on keeping the site alive and performing well.


Think about that for a moment - they have a *single* site, and at least 40 employees dedicated to making it work.  Keep that in your head as you read the rest of this article.


On August 11, Reddit fell over for about 3 hours.  Thats clearly a really big deal, as when reddit isn't available, reddit isn't making money - at all.  Their website *is* their mission critical system.  They posted a [great post mortem of the issue](https://www.reddit.com/r/announcements/comments/4y0m56/why_reddit_was_down_on_aug_11/) on their site for all to read.  Overall, the issue came down to human factors:

> Since autoscaler reads from Zookeeper,** we shut it off manually** during the migration so it wouldn’t get confused about which servers should be available. It **unexpectedly turned back on** at 15:23PDT because our package management system noticed a manual change and reverted it. Autoscaler read the partially migrated Zookeeper data and terminated many of our application servers, which serve our website and API, and our caching servers, in 16 seconds. [emphasis mine]

I found this root cause fascinating, because its pretty clear how to prevent it.  Manual tasks are bad.  The most striking thing to me is how this maps into well understood design patterns.


If you haven't [read the 12 Factors](https://12factor.net/), you almost certainly should.  Specifically, [Factor 12](https://12factor.net/admin-processes) stuck out to me here: 

> One-off admin processes should be run in an identical environment as the regular long-running processes of the app. They run against a release, using the same codebase and config as any process run against that release. Admin code must ship with application code to avoid synchronization issues.

In short, this factor is saying that your admin processes (like disabling autoscaler systems during an upgrade) should be a regular part of the code base of the app, and not something that you just 'wing it'.  In other words - carefully following the 12 factor design principles helps avoid these kinds of issues.

Now, to be clear, everyone makes mistakes, and I'm not suggesting Reddit's engineering crew masssively screwed up - this stuff is really hard.  But it does show that common design patterns have value in being able to learn the lessons that others have paid for.

---

Now, here comes some opinion.

Reddit has 1 application.  Reddit has 50+ engineers, based in San Francisco, where some of the best engineering talent exists.  They are known to embrace and have significant experience with DevOps methods.  They've chosen to build their own platform and management tools because thats what makes them different.

Most of Dell & EMC's customers have many applications (dozens, or hundreds even).  They have maybe a dozen or 25 engineers spread across those dozens of applications.  They aren't necessarily where the talent flocks to.  They aren't experts in management platforms or tooling.

My argument is this...if even Reddit, with all their advantages, doesn't get it right [beyond 2 nines availability](https://uptime.com/reddit.com),what chance does the average smaller, less focused company have?  I'd argue those companies should *not* be trying to implement platforms and follow the 12 factors on their own.  Those companies should be using an existing platform that performs these kinds of management tasks in a predefined, known good way.  They should be using a *prescriptive* platform that guides the right decisions rather than *composing* a bunch of tools together and hoping they can do better than Reddit.

In other words, I think companies that aren't Reddit, Twitter, Google, Uber, Facebook, etc should be using [Cloud Foundry](https://www.cloudfoundry.org/) rather than Linux+KVM+CoreOS+Mesos+Marathon+Docker, etc.   I think *presciptive platforms* are a better choice than *composable platforms* for everyone that isn't Reddit.

