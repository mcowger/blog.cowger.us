---
layout: post
title: Blog Workflow with Jekyll
author: mcowger
categories: [blog,github,markdown,code]
published: true
comments: true
---

Ever since my post about moving my blog to GitHub Pages and Jekyll, I've had a couple requests for my 'workflow', or how I use this on a daily basis.

In my opinion, its pretty simple, and easy.

I generally start by opening my text editor to get started writing.  I generally prefer to use a light weight text editor for this sort of thing (rather than a dedicated Markdown editor, for example), and I've found that  [Atom from Github](https://atom.io/) is actually quite good.  Its fairly light weight, highly configurable and fast.

I have a couple plugins that I use to make it better.  Specifically, I use the [Markdown Preview package](https://github.com/atom/markdown-preview) to make it super easy to preview what I write as I go.  One of the things I like is that it handles the YAML Frontmatter that Jekyll uses for meta data well - better than most others.  It looks something like this:
![fullscreen]( {{site.baseurl}} /images/2015/02/fullscreen.PNG)

The second plugin I use is the Markdown Writer package, which adds some convenience functions like making it easier to insert links, images, tables etc.
![writer]( {{site.baseurl}} /images/2015/02/writer.PNG)

Lastly, I make sure that the YAML Frontmatter for the post is set to `published: false` so that I dont accidentally post something half complete.

Once I have the post written and images inserted, I save the post in the Jekyll `_posts` directory.  

From there, the rest happens at the terminal.  I have my entire repo checked out where the rest of my Git repos live (for me, `~/Projects/blog`).  My prompt ([generated with Prezto](https://github.com/sorin-ionescu/prezto)) shows me that I now have untracked files (a red dot in the prompt):
![terminal]( {{site.baseurl}} /images/2015/02/terminal.PNG).  The next step is to add that new post to the repo with `git add _posts/filename.md`.  My prompt then adds a green dot instead, indicating I have uncommited changes to the repo.  I commit the changes with `git commit`.

My last step is to preview it with full rendering by running `bundle exec jekyll serve` and visiting the link presented:

![terminal]( {{site.baseurl}} /images/2015/02/bundle.PNG).

Once I'm certain it looks OK, the final step is to commit and push:

![terminal]( {{site.baseurl}} /images/2015/02/push.PNG)

And with that, its live!
