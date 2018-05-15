---
layout: post
title: vCloud Air Is A Terrible AWS Alternative - And That's Good
author: mcowger
categories: [blog,vcloud,air]
published: true
comments: true
---


It really is a terrible AWS alternative.  The focus of the [vCloud Air](http://vmw.re/1y9bClv) is an entirely different design center.  How can you tell?  Easily, through a variety of lenses.  Just 2 to start with:

1. Look at the technology used.  The compute is standard off-the-shelf hardware, but everything is dual pathed, dual power supplied, etc.  The network is all high end gear.  The storage is all very high end hardware designed for resiliency.
2. Look at the SLAs.  What SLA will AWS sign?  Right around 3-9s.  Microsoft Azure?  3-9's.  vCloud Air?  5-9s.  Why?  Because its built differently, and can actually achieve that.

What does that buy you?  It means you can run critical COTS applications (e.g. ones you dont control and aren't like Netflix) without the ~1% failure rate we often see in AWS deployments.  Thats stuff like Exchange, SAP, etc.

Now, whats the downside to that (because everything in engineering is a tradeoff)?  Its very likely more expensive per GHz/MB/GB.  You can't deploy 1000 VMs in 4 minutes like AWS could.  It doesn't have a native PaaS like AWS does.

But thats OK, and even a good thing.  [vCloud Air](http://vmw.re/1y9bClv) fills a spot in the public cloud that wasn't being properly served.  People have wanted someting that was as accessible as AWS (with a similar cost model), but as reliable as a proper mission critical environment.

vCloud Air has finally delivered on that.  With their new On Demand version (the codename for this was 'Praxis').  Its

* easy to use (which we already knew from the previous, non On-Demand version)
* reliable (again, something we already knew)
* clearly affordable...a single, basic, VM is within 30% of the price of one from AWS (comparing to a [t2.micro](http://aws.amazon.com/ec2/pricing/) here), with far better performance guarentees at [$0.06/hr](http://vcloud.vmware.com/service-offering/pricing-calculator/on-demand)

So its a solid offering.  And heck, they give [everyone $300 credit ](http://vcloud.vmware.com/service-offering/virtual-private-cloud-ondemand)right now, which is is 6 months of that entry level VM I mentioned above.  Easily enough to run a blog, etc.

So is it a terrible alternative to AWS?  Yes - it has wildly different cost, performance and resiliancy models.  But thats why its great - it fills a niche that wasn't filled by AWS or really any others, in my opinion.


_disclosure: the [vCA](http://vmw.re/1y9bClv) team gave me early access to the product to test, but does not exercise control over this post._
