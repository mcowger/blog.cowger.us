---
publish_date: 2017-04-12 00:00:00 -0700
post_title: A Cloud Foundry Story

---

A Cloud Foundry Story - Idea to Production in 90 Minutes

One of the best parts of what I do is making demos - they are an awesome combination of playing with the latest technology, making it do something cool (but understandable) and not having to support it for more than a couple months.

This year, for EMCworld and VMworld, I wrote a demo called ['Vasco da Gama'](https://github.com/mcowger/vascodegama/tree/vmworld-branch) that was used in a number of places, including Chad's blog, his session at VMworld and at the EMC booth.  In short, this demo captures pictures with the #vmworld hashtag displays then to a user - it ran on vCloud Air using [Pivotal Cloud Foundry](http://pivotal.io/platform) as the platform.

![fit](http://virtualgeek.typepad.com/.a/6a00e552e53bd2883301b8d14f0a8c970c-pi)

However, during the show we discovered a 'bug'.  It turns out that some ne'er-do-well on the internet realized that the [#vmworld hastag was trending](https://twitter.com/search?q=%23vmworld&src=typd) (as it is likely to do with 23K attendees), and decided that it would be a good tag to post some, ahem, NSFW pictures with.

This became a problem, as the demo simply blindly accepted these photos and displayed them to attendees.  For some reason, this made the EMC marketing people a touch nervous, and they ~~demanded I~~ asked me to fix it.

The immediate fix was to wipe the database, and this was easy as the application has a [microservice architecture](http://microservices.io/patterns/microservices.html), and a specific service for adminsitrative functions called `scaler`, one of the 12 factors that make cloud native applications successful.  So, once the database was wiped of any concerning images (using the `/clear` REST endpoint) and had newly collected a few innocuous images, the next step was to shut down collection of new images, so that the demo would stay somewhat static, but remain available.  Again, because of the [microservices architecture](http://microservices.io/patterns/microservices.html), it was a simple matter of stopping the `watcher` service with one simple command (`cf stop watcher`).  From there were were safe to show the demo again, and I had time to play with solutions to the problem.

We had a couple lessons learned here.

> Lesson 1: Microservice architectures make it easy to indepdently control application behaviors without losing the entire application.

and

> Lession 2: Common administrative functions should be easy to perform without resorting to manual database or other tasks.

Now that we had some breathing space, my colleague [Luke W](https://twitter.com/luke4oss) pinged me with an idea - what about categorizing based on URL?

We met a few minutes later above the EMC booth to discuss.  In short, he had noticed that most of these NSFW tweets had URLs in them (to direct unwary clickers to a signup site).  He had also noted that it seems that the team over at BlueCoat has a public REST API for querying the category of given websites, and [there was some sample code around on GitHub.](https://github.com/idiom/IRScripts/blob/master/urlinfo.py)

So from there, we got started with coming up with a solution.  We built a function that would extract all the URLs from a given tweet and check them against this database:

{% highlight python  %}
def check_porn(check_url):
worker_logger.info("Checking URL: {}".format(check_url))
if not check_url:
return False
response = requests.post(url="http://sitereview.bluecoat.com/rest/categorization/", data={"url":check_url}).json()

    if re.search("adult|mature|sex|porn|extreme",response['categorization']):
        print("PORN FOUND")
    
        worker_logger.warn("Found porn: {}".format(check_url))
        worker_logger.warn("Category: {}".format(response['categorization']))
        return True
    worker_logger.debug("No Porn Found")
    return False

{% endhighlight %}

And this was a _great_ start, and seemed to catch much of the problem from this particular botnet/sender.  The downside to this model is that we are now doing a relatively expensive external check for many tweets, which slows down processing.  We absorbed that degradation of throughput by increasing the number of workers processing these tweets.

> Lesson 3: Microservices and stateless 'workers' allow you to absorb drops in individual performance gracefully.

We deployed this code at about 6:02 PM local time - _in other words 90 minutes after the idea_.  This is the promise and achievement of something like cloud foundry and cloud native applications - idea to production in 90 minutes.

> Lesson 4: Platforms that make deployment and testing easy are critical to rapid development

Lastly, we noticed something else while diving through the [Twitter Streaming API](https://dev.twitter.com/overview/api/tweets): they had solved the rest of this problem for us.  See - not all inappropriate tweets have a URL, and even some of those might be uncategorized.  But Twitter knows this, and there is a specific field in every tweet that is `True` if they deem the tweet 'sensitive:

{% highlight json %}
"possibly_sensitive":true
{% endhighlight %}

As a result, we felt safe droppping every tweet with that property set to true before even performing our more expensive BlueCoat-based check:

{% highlight python %}
if tweet\['possibly_sensitive'\]:
raise Exception("Sensitive Tweet")
{% endhighlight %}

We pushed this change about 30 minutes after the first, and found that for the rest of the show, we didn't have a single objectional tweet displayed on the board for the remainder of the show.

> Lesson 5: Blue/green online deployments make it possible to trying multiple different solutions to a problem quickly and safely.

And did I mention, this all happened while the teams down below were actively demonstrating the application and well as pushing updates to the `viewer` service (that displays the images to the user)?

Writing this demo was a ton of fun, but the 'problem' faced during the show made it even more clear that tools like Cloud Foundry and practices like the 12 factors lead to easily managed applications that fail gracefully and can be improved on the fly.