---
layout: post
title: "Blog refined"
date: 2020-04-21 11:00
author: scooletz
permalink: /2020/04/21/blog-refined/
image: /img/2020/04/blog-refined.png
categories: ["blog", "personal"]
tags: ["blog", "personal"]
whitebackgroundimage: true
---

This is the first entry on my new old blog. As you can notice, the layout is more less the same. There's a top menu, a sidebar and a list of posts. What was the reason to migrate from Wordpress then? What tools are behind the new one? What has changed?

### The old blog

The old blog was using _Wordpress_ hosted by wordpress.com. The pricing plans are affordable if you want to just run the blog. This is a good option for sure for  a technology agnostic person, that wants to have a blog that provides some value. For a person that knows what html, css and js is, there are other options like Ghost, Jekyll and all the rest static site generators.

I think the most important thing that pushed me to change the medium (pun intended) for my blog, was the disability to do more things without paying a lot more. As always, pricing is a sliding scale and this scale for me was a bit too much. The other thing was for sure influencing the speed of the page and being able to configure things. Again, one may follow the path that _toggling is fun_ and that big enough plan will enable you to configure everything. On the other hand, the new static sites generators allow you to write a line of code that will impact the site generation. This might be easier, faster and much more powerful, especially if the person knows the basics of a programming flow control like conditions, loops, etc.

The first part of migration was extraction of the stuff that I wrote during last 10 years.

### Extraction

The extraction from Wordpress started with the content extraction. This is done in two separate steps:

1. The extraction of the text content, including: posts, pages, comments
1. The extraction of the media (images)

The first step produces a huge single xml, that includes all the data. The xml schema is standardized, but it has some drawbacks.

The first issue is the way comments, posts are related. The identifiers are used, so the items are separate nodes, referring to each other by identifiers. This requires keeping a mapping, not only between the posts and comments but also between comments as they point to each other with _parent id_ property.

The other thing is the quality of the produced markdown because of (probably) my clumsy edits in the wordpress itself. I used code found on Github, but even with a bumped up _html to markdown processor_ the output was miserable:

1. lists were separated by multiple lines, hence creating a messy structure
1. headers were not followed by a double new line
1. posts were ended by multiple new lines
1. bold was used instead of proper headers

Fortunately, the majority of these issues can be solved with a well written regular expressions which I did. Now blink twice while you can, because it's a pure regex nightmare

```csharp
static readonly Regex OrderedListSpaces = new Regex(@"\r\n\d\.[^\S\r\n]{1,3}", Flags);
static readonly Regex UnorderedListSpaces = new Regex(@"\r\n\*[^\S\r\n]{1,3}", Flags);
static readonly Regex BoldAsHeaders = new Regex(@"[\r\n]+\*\*(.*)\*\*[\r\n]+", Flags);
static readonly Regex OrderedListNewLines = new Regex(@"((?:\d\.[^\S\r\n].+[\r\n]+)+)", Flags);
static readonly Regex UnorderedListNewLines = new Regex(@"((?:\*[^\S\r\n].+[\r\n]+)+)", Flags);
```

and a few more. Yes, this required coding but was much more efficient as I rerun the importer several times solving issues one by one.

With this sanitization, the output was almost perfect (around 90% of posts required no additional clean up). To be able to tell whether I reviewed a blog entry I added a flag `imported: true` which showed that the post was imported but not reviewed.

The comments! They were extracted with the metadata and put into a separate place. I didn't have a lot of them, but still didn't want to loose any of them when migrating. After all, it's people's time invested in my blog!

The part about the media was easy. It was just a single directory with all the pictures.

### Setting up the repositories

From the very beginning I had this idea to keep posts separated from the page itself. This would allow me and the readers to see a simple repository with posts to contribute to when/if needed. On the other hand, it would keep the ceremony (styling, layouts, javascripts, integration) separated. As a reader and a contributor, you should not be bothered with it. 

To make this happen, all the posts excluding comments, images etc. were moved to [blog.scooletz.com-posts](https://github.com/Scooletz/blog.scooletz.com-posts). I added a simple `readme.md` to describe additional work that needs to be done when contributing (adding tweets for now is a manual work). The comments with the metadata (like emails) were put with the ceremony part, in a separate private repository.

How these two are combined? This is a good question.

### Blending it together

The ceremony part has a git submodule tracking the posts' `master` branch. The page is built with the regular Github Pages build, that pulls the submodule and puts it in a regular Jekyll collection of `_posts`. With this, it's really easy to build site locally. To see how a post is rendered, I  can either add it locally or reset a module to a PR branch in posts repo. Again, with markdown behind posts, it's really hard to mess things up. If there's something terrible, it will just not build, so again, no risk here. My assumption is, that with markdown, I should be able to create posts faster, with less ceremony and provide longer write-ups without dealing with all the interruption from the online editor (not to mention some WYSIWYG or others).

### The page itself

The page itself is a heavily modified template. It uses Bootstrap as I'm not a css pro to handle all the paddings, etc. What I did to minimize the overhead though is following:

1. using `jekyll-coffeescript` plugin to enable Sass style compilation
1. switching from the precompiled cdn css version of Bootstrap to Sass based
1. toggling all the components that are not needed, explicitly including only these that I know that will be used
1. removing a lot of [themes from Bootstrap](https://getbootstrap.com/docs/4.1/getting-started/theming/) using `$theme-colors: map-remove($theme-colors, ...);`

This allowed me to host a unified single css blend of my css and Bootstrap at the same time.

The rest of the page is simple but includes a few interesting aspects:

1. comments
1. my lovely [ConvertKit](https://scooletz.com/links/convertkit) integration
1. awesome [Integromat](https://scooletz.com/links/integromat) scenarios for... integrating things.

Comments are for now rendered as readonly. I do this to respect authors and their point of view shared in comments. The ability to add comments should come quite soon. I'm looking for some options now.

The [ConvertKit](https://scooletz.com/links/convertkit) that I use for my mailing list, is finally integrated in a way, that I can easily pop up a form to a user interested in obtaining something and joining to my mailing list. With Wordpress it would require either self hosting or paying a lot for being able to add a simple script. This is not the case anymore. Again, I'm not switching to anything else, like never ever, because ConvertKit is really good.

The last but not least is the mighty [Integromat](https://scooletz.com/links/integromat). I use it to extract some metadata from the ATOM feed of my blog and publish entries to my social media accounts, including delayed re-shares. As a software engineer I'd never thought that `no-code` tools can be so cool. If you haven't tried it and you have something to integrate with, please try out [Integromat](https://scooletz.com/links/integromat).

### The migration of DNS

When working on the new site I moved my DNS to [Cloudflare](https://www.cloudflare.com). This was done to use their intelligent caching and [Page Rules](https://www.cloudflare.com/features-page-rules/) that enabled a simple migration for a few of my urls. Additionally, I started playing with the edge computing of [Cloudflare Workers](https://workers.cloudflare.com/) but this is not production ready yet.

### The tooling

To summarize all the tooling that I used, from the beginning till the end:

1. an export from Wordpress, both media and content
1. an html to markdown with a lot of rexes to fix up my nasty formatting in Wordpress
1. two repositories:
   1. [a public one with posts](https://github.com/Scooletz/blog.scooletz.com-posts)
   1. a private one with the page itself based on [GitHub Pages](https://pages.github.com/) including the comments data with metadata
1. [Jekyll](https://jekyllrb.com) for site generation
1. [Bootstrap theming](https://getbootstrap.com/docs/4.1/getting-started/theming/) to make css smaller and faster
1. [ConvertKit](https://scooletz.com/links/convertkit) as still the best mailing software I ever used
1. [Cloudflare](https://www.cloudflare.com) for CDN and routing a few problematic pages

### Summary

I hope this posts helps in a few ways. It shows my story/way of migration (no link was broken or made inaccessible!). It shows the tooling behind, so maybe you learnt a thing or two about this pallette. Finally, it shows what can be done with a bunch of tools and that sometimes combining things brings so much more than one huge solution for a problem.

Frankly speaking, I really enjoyed writing this down in my markdown editor. It was, smooth and efficient. It's time to push it and let it build!

I know the comments are not available now, but below you have all my social media accounts. If you find it interesting, do not hesitate to contact me or simply mention me when sharing this post.

See you soon!
