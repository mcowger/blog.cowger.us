---
layout: post
title: Blog Movement - Jekyll & GitHub Pages & Disqus
author: mcowger
categories: [blog,github,markdown,code]
published: true
comments: true
---


I've never really liked Wordpress.  Its heavyweight (heavier certainly than I need, and supports a thousand plugins I'll never use).  Its [written in PHP ](http://webonastick.com/php.html).  It [requires MySQL](http://grimoire.ca/mysql/choose-something-else) (which no one should be using).  It requires me to run my own server and do my own patching/maintenance, which I'd rather not do.

So I had been looking for an alternate platform, and a [post from Scott Lowe](http://blog.scottlowe.org/2015/01/05/blog-migration-complete/) inspired me to try [the Jekyll route.](http://jekyllrb.com/)

So as of right now, you are looking at a static version of my blog, rendered by [Jekyll](http://jekyllrb.com/), hosted by [GitHub Pages](https://pages.github.com/) and without PHP or MySQL.  And I did it in less than a day, with all content in place, while maintaining all comments, even.

So how did I do it?

1. I used the Jekyll [Wordpress Importer](http://import.jekyllrb.com/docs/wordpress/) to access my MySQL database and pull out of it all my posts and metadata.  The nice thing about this method is that its quite fast, and properly populates all the [YAML Front Matter ](http://jekyllrb.com/docs/frontmatter/) required by jekyll to properly create permalinks later.  This created all ~67 posts as Jekyll pages.  The downside was that much of the HTML formatting is kept in the file as well.   While this is helpful, because it makes things display properly, it does mean that half my site is formatted in HTML and not with converted Markdown.  I may make an effort later to convert these to Markdown using something like [Pandoc](http://johnmacfarlane.net/pandoc/) or [TextSoap](http://www.unmarked.com/textsoap/).
2. Next I had to find a way to get my images.  I ended up creating a folder next to my `_posts` folder in my blog repo (remember, this is all kept under version control with Git/Github) called `images`.  I then copied the entire contents of my `wp_content/uploads/` folder from my server to `images`.   The last step was to convince all my blog images to load from the new Jekyll hosted location instead.  I did this with a little perl regular expression action: `perl -p -i -e 's/http\:\/\/www.exaforge.com\/wp-content\/uploads/\{\{ site.baseurl \}\}\/images/g' *.markdown`, thus replacing all instances of `www.exaforge.com/wp-content/uploads` with `site.baseurl` in the files.  When Jekyll goes and generates the site, it will replace `{{ site.baseurl }}` with the actual location of my blog.  Nice!
3. My third step was to migrate all my comments, as those were critically important to me.  I decided to do what nearly everyone else did, and migrate my comments to the [Disqus platform.](https://disqus.com/)  This was a 2 step process: 
    1. I did this by exporting my comments using the [method they describe on their help pages](https://help.disqus.com/customer/portal/articles/466255-importing-comments-from-wordpress), which went swimmingly and only took about 10 minutes.
    2. I added the Disqus universal code to a proper block in my HTML templates [using their recommendations](https://help.disqus.com/customer/portal/articles/472138-jekyll-installation-instructions).
4. With content, images and comments out of the way, I had a pretty decent looking site.  But I wanted something that looked a little different than everything else.  I found a nice [looking Jekyll template on GitHub](https://github.com/dbtek/dbyll) that worked well, and cloned it, copying my `config.yaml` and `_posts` folder into it.  The only major change (besides stuff like adding my github account) to the `config.yaml` I made was changing the permlink style to match what it had been on wordpress, so my permlinks would still work.  I changed it to `permalink: /:title/`
5. My (nearly) last step was to prepare to push the site to GitHub pages.  I did this by resetting the cloned template from above to push [to my own GitHub repo](https://github.com/mcowger/mcowger.github.io) I had created, and adding my new files. Thats simple set of  git command: `git add * && git remote set-url origin https://github.com/mcowger/mcowger.github.io.git`.  Now any time I did a `commit` then `push` it would automatically be loaded into GitHub pages.  
6. After my first push, my site was available at http://mcowger.github.com.  But thats not quite what I wanted, because I it to be at http://www.exaforge.com like it had been before.  First, I had to tell GitHub to serve it from that location by creating a file in the root of my repo called `CNAME` with contents of `www.exaforge.com`.  After adding, committing and pushing that, I made the proper changes to my DNS settings (I use [CloudFlare](http://www.cloudflare.com)) to make www.exaforge.com a `CNAME` to `mcowger.github.io`.

After all that was complete (it took a couple hours, including cleaning up some layout errors on a few posts), I was done and ready to go.  Cool stuff!

My next post will likely go into my editing, testing and pushing workflow.