---
layout: post
title: CloudFoundry Summit Quick Take
author: mcowger
categories: [cf,cloud,foundry,cloudfoundry,pivotal,conference]
published: true
comments: true
---

I thought I'd just write a quick review of the Cloud Foundry Summit I attended last week.  Note: Everything was recorded and will be posted over the next couple weeks, and when that happens I will post another set of links to the best talks.

#Major Announcements:
* 1400 attendees (+50% from last year).
* [JP Morgan Chase joins the CF Foundation](http://leverhawk.com/five-takeaways-from-the-jpmorgan-chase-paas-announcement-20130308211)
	* The second major financial to join
* New Relic joins the federation
  * The first major SaaS provider I've seen.
* Diego:  Plan for GA release in Q3 of this year.
* [HP donates thousands of lines](http://lists.cloudfoundry.org/pipermail/cf-dev/2015-May/000023.html) - Windows fully supported, in effect:
  * WinDEA - allows using Windows as a DEA, and therefore .NET and other windows applications on Cloud Foundry.
  	 * Already in the incubator, targeting Q3 GA
  	 * Supports Windows 2012
  	 * Uses IIS isolated sites
  * SQL Server Service Broker (SQLServer 2012/2014)
      * No restrictions on nature of backend databases (e.g. AAG's allowed).
      * Requires 'contained' databases for security.
  * VisualStudio Plugins
      * Support for deploy & management to CF from within VS
      * Mac & Windows
      * Free
      * VS 2013 and VS 2015
      * Local compilation or server side compilation supported.
  * "exe" buildpack
    * Similar to 'null' buildpack, allows push of raw '.exe' for execution in DEA.
* EMC opening Dojo in Cambridge in Summer
* IBM opening Dojo in RTP later this year
* GE opening an 'industrial' dojo.
* Microsoft [announces support for CF on Azure ](http://azure.microsoft.com/blog/2015/05/07/want-to-learn-more-about-cloud-foundry-on-azure/)
* VMware's Kit Colbert [announces a Photon-based stemcell](https://twitter.com/wattersjames/status/598167559712604161), making a very small, non-Ubuntu stemcell a possibility (*ed: which is hugely positive for everyone that doesn't like Ubuntu for production, which includes every enterprise ever, and, well, me :)*
* [Mirantis and Pivotal announce alliance](http://www.forbes.com/sites/benkepes/2015/05/11/mirantis-and-pivotal-ink-a-deal-part-brand-building-part-customer-value-add/)


#Videos
All the videos for [CF Summit have been posted](https://www.youtube.com/playlist?list=PLhuMOCWn4P9g-UMN5nzDiw78zgf5rJ4gR) (that was quick!).

Here are some of my picks:

* [Diego Update](https://www.youtube.com/watch?v=SSxI9eonBVs&list=PLhuMOCWn4P9g-UMN5nzDiw78zgf5rJ4gR&index=32)  This presentation was the single best of the conference in my opinion.  The content was phenomanal, and Onsi's presentation skills are some of the best I've ever seen.
* Of course, m[y session with Rags ](https://www.youtube.com/watch?v=QOz1UOf6MdU&list=PLhuMOCWn4P9g-UMN5nzDiw78zgf5rJ4gR&index=39)from EMC{code} on 12 factor apps.
* [Let Diego Manage your Docker Application](https://www.youtube.com/watch?v=tQC8vz1dedI&list=PLhuMOCWn4P9g-UMN5nzDiw78zgf5rJ4gR&index=58)...title says it all.
* [The process of becoming a contributor](https://www.youtube.com/watch?v=GTAqlblhELo&list=PLhuMOCWn4P9g-UMN5nzDiw78zgf5rJ4gR&index=54)...using the CF Dojo.
* [The Road to Persistence.](https://www.youtube.com/watch?v=3Ut6Qdd2FHY&list=PLhuMOCWn4P9g-UMN5nzDiw78zgf5rJ4gR&index=55)..again, title says it all.
* [EMC's Brian Gallagher's talk](https://www.youtube.com/watch?v=dHgupDLiS2A&list=PLhuMOCWn4P9g-UMN5nzDiw78zgf5rJ4gR&index=43)...hints at where EMC is going.
* Nerding out on Garden, Docker, other CF underpinnings...[Docker w/ Garden](https://www.youtube.com/watch?v=x_Zshlq4vgE&list=PLhuMOCWn4P9g-UMN5nzDiw78zgf5rJ4gR&index=31), [Logging and Metrics](https://www.youtube.com/watch?v=jTxnCV7wjeA&list=PLhuMOCWn4P9g-UMN5nzDiw78zgf5rJ4gR&index=29) and [Life in the Trenches.](https://www.youtube.com/watch?v=c07WxRw30Vs&list=PLhuMOCWn4P9g-UMN5nzDiw78zgf5rJ4gR&index=6)


#Resources:

##Blog Posts:
* [Visual Studio tools ](http://t.co/0PGaCdbc1u) from HP
* [Altoros overview](http://blog.altoros.com/cloud-foundry-summit-2015-day-one.html)

##Slides / Sources
* Matt Stine's slides on microservices: https://t.co/6LzQTkZCmv
* Slides on Lattice: https://t.co/p28MKttB2S
* 10 common errors when pushing to CF: http://www.slideshare.net/greensight/10-common-errors-when-pushing-apps-to-cloud-foundry?qid=f4f55081-5609-47ad-910c-05ba794b0612&v=qf1&b=&from_search=2
* How to organize a CF User Group: http://www.slideshare.net/DanielKrook/findingandorganizingagreatcloudfoundryusergroup
* Wikibon's take: http://siliconangle.com/blog/2015/05/11/confused-about-paas-wikibon-can-help/
* Kroger's story: http://www.bizjournals.com/cincinnati/news/2015/05/04/heres-how-technology-is-changing-your-kroger.html?utm_content=14852583&utm_medium=social&utm_source=twitter
