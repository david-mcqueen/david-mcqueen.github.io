---
layout: post
title:  "Hello, World!"
date:   2018-11-22 22:13:00 +0000
categories: jekyll update
---

# As is tradition.

Each time I've started a blog, and there have been a _couple_, I've opted for more of a 'traditional' platform. I've used wordpress, both the self hosted / deploy method, and the pre-built 'one-click deploy' solutions. I've used Blogger. I've used ghost.io. I've probably used another long forgotten platform, of which there are plenty.

All of these had a fairly similar premise: A browser based word editor within an 'Admin' portal, followed with a corresponding 'user' website to navigate each of the posts. They stored each of the posts and comments within a database - which makes sense as posts want to be created / edited & persisted - but it could possibly be a little too much. If the site is only going to be serving up the same content to each user why do we need it to be in a database? Conversley we don't want to be writing a static html page and handling all of the links etc manually.

I must admit, I had never given it ~~too much~~ any thought before now - it is just the way it is.

That was until I came across [Jekyll](https://jekyllrb.com/).

With Jekyll all of the posts are written in markdown, they are then transpiled to static HTML which are in turn served up to the site visitor. No need for a database. No need for any fancy comments. No need for the kitchen sink. Just a simple blogging platform.




## Structure

All posts are written within the __posts_ folder with the format _YYYY-MM-DD-post-name.markdown_ which, when transpiled, end up within the __site_ folder. As this cheeky image below shows;

![Folder Structure](/assets/images/hello-world/folder-structure.png)


The top of each post needs to contain some meta-data which gives Jekyll some information needed for the home page.

```
---
layout: post
title:  "Hello, World!"
date:   2018-11-22 22:13:00 +0000
categories: jekyll update
---
```

With a markdown post written, simply running a one line command within... command, will transpile the markdown into HTML and serve it up on http://localhost:4000

```
bundle exec jekyll serve
```

We have our blog!


> "_Markdown content goes in. HTML webpages come out._"

_Me - 2018_


## Whats next?
This started as a simple "Hello, World!" and has evolved into a bit of a fly-by overview / explanation of Jekyll itself. No bother. We have a bit of insight into writing posts and ending up with the nice formatted post which is fundamentally what you want from a blogging platform.

### The rest of the Owl
Other than the knowledge of starting with the user input of _markdown_ and ending up with the Jekyll output of _html_ we don't really know much. What about the rest?

![Owl](/assets/images/hello-world/owl.jpg)

Watch this space! In a subsequent blog post I'm planning on digging into what Jekyll is doing to get from markdown to html. What is Jekyll doing, so we don't have to?