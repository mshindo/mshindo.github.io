---
date: 2023-03-10 07:09:00+09:00
layout: post
title: Migration from WordPress to Jekyll
image: '/images/robinson-greig-HrnAxAUwle8-unsplash.jpg'
tags:
- Computer and Networking
language:
- English
keywords:
- Wordpress
- Jekyll
- GitHub Pages
---
I changed my blog platform from WordPress, which I have used for a long time, to Jekyll (deployed on GitHub Pages).

## Background 

My blog platform has been WordPress on a virtual machine running on an ESXi server at home. However, with the renovation of my apartment, I had to move to a temporary residence. Consequently, the WordPress environment also had to be moved somewhere, which made me write this blog article about such a transition.

## Strategy 

As a migration strategy, there were two ways I could take: 

1. Run WordPress on a public cloud somewhere and use it temporarily.
2. Stop using WordPress and migrate to something else. 

Idea #1 was the easiest. Still, some challenges with WordPress have suffered me, so I decided to take Idea #2, moving my blog to another platform.

Although WordPress is an excellent blogging platform, it has some downsides, as shown below:

- It has a large attack surface; PHP and the plugins require an upgrade so frequently.
- WordPress is a very "sticky" system because it relies on DB, making it hard to move to another platform.
- I feel my WordPress environment is becoming more "polluted" as I use various plugins.

One solution to alleviate these challenges is to use the so-called **Static Site Generator, SSG**, among which Gatsby, Jekyll, and Hugo are probably the most famous. I could have used any of them, but considering the platform on which to deploy it, GitHub Pages seemed more likely to be stable as a service than other choices like Vercel or Netlify, so I decided to go with Jekyll, the default static site generator for GitHub Pages (another reason was that Jekyll is written in Ruby, which I'm reasonably familiar with).

## Content Migration

Many people have tried transitioning from WordPress to Jekyll, and thanks to those pioneers, some tools are already available. First, I tried converting from WordPress to Jekyll using this [article](https://dev.to/rupeshtiwari/importing-wordpresshan-or-blogger-blogs-to-jekyll-blog-mpg) as a guide. This procedure allowed me to download my WordPress blog content (articles and images). But even though the article I followed says that blog posts in WordPress are downloaded as Markdown files, they're downloaded as HTML files. It's not a fatal issue because Jekyll can handle HTML files just fine. Still, I wanted to use Markdown for all my blog posts for brevity and consistency.

After some struggle, it seemed challenging to download the contents as Markdown files with this procedure, so I gave up and tried the alternative introduced in another [article](https://taroyabuki.github.io/2018/08/18/switching-to-jekyll-from-wordpress/) (sorry, it's only in Japanese.) With this method, the text files were downloaded as Markdown as expected, but it didn't download the images. So, I copied the image file directly from `/wp-content/uploads/` on my WordPress server via the `scp` command. (I could do this because I ran my own WordPress instance with full access. However, if you use a WordPress hosting service, you can't do it. If so, you can try the procedure I mentioned first.)

At this point, we have all the content that was on WordPress. To get the migration completed, we need a few more tweaks. The most significant task still left is changing the link to an image file. The Markdown file downloaded in the way described above still contains a link to the original WordPress image location, which must be rewritten to reference the local file. I did this with the help of Visual Studio Code's regular expression substitution feature. It's very convenient because you can replace the target word on multiple files all at once (I would have given up this task if this feature wasn't available).

Having completed all the tasks described above, you can now use Jekyll to generate and verify a site.

## Jekyll theme selection

Jekyll has many themes. You can find them at the following sites, for example, and choose one you like:

- [http://jekyllthemes.org/](http://jekyllthemes.org/)
- [https://jekyllthemes.io/](https://jekyllthemes.io/)
- [https://jamstackthemes.dev/ssg/jekyll/](https://jamstackthemes.dev/ssg/jekyll/)

Unfortunately, there are so many excellent themes that it's hard to choose. To shortlist it from a long list of possible choices, I used the following criteria: 

- Suitable for Blogs
- The design is simple (not showy)
- Ad-Free
- Articles are listed and paginated properly
- The image at the top of the article (a.k.a. "Featured Image" or "Eye Catch Image") appears as a thumbnail
- OGP compatible

The theme that caught my eye that seemed to fulfill these requirements was [Menca](https://jekyllthemes.io/theme/menca-blog-jekyll-theme). The only downside of this theme may be that this is not free ($49 as of this writing), while there are so many free and excellent themes. But Menca came closest to what I wanted, so I purchased it. I think it was well worth the price.

I am satisfied with Menca mostly, but there was only one point that was not to my taste; the article without a featured image didn't look very nice. So I made a simple change, like [this one](https://github.com/mshindo/mshindo.github.io/commit/28af0763e79bb124acc1e377941f9267b984dc3a), to allow you to specify which image appears by default in articles that don't have a featured image with the following configuration in `_ config.yml`: 

`default_image: "/images/whatever-image-you-like.jpg"`

At this point, the migration is almost complete. The only tweaks you may need to make are:

- Remove the unnecessary Front Matters
- Map the 'category' in WordPress to a 'tag' in Menca
- Manually embedded YouTube videos

Another slight inconvenience of Markdown is the lack of centering capability that the HTML has. I applied the method described in this [article](https://choose0or7.github.io/posts/ja/center-text-and-image-in-markdown) (sorry again, it's in Japanese) to where the centering is absolutely needed.

At the moment, there are a few things we have yet to address.

- ~~Embed Twitter Tweets~~ (addressed on 2023/02/22)
- ~~Table in Markdown~~ (addressed on 2023/02/26)
- ~~Search function does not work with Japanese~~ (addressed on 2023/03/01. It turned out to be that the Japanese text was the cause. Rather, it was because the content was too big. I resolved this problem by removing the content from the search criteria and adding keywords instead.)
- SEO measures
- ~~Links not prominent~~ (addressed on 2023/02/20)

But these are not a rush for now, so I hope I can deal with it in the future.

## Conclusion

Along with moving to a temporary residence, I also moved my blog set up from WordPress to Jekyll deployed on GitHub Pages, which had sort of distressed me for a long time. The transition took quite a bit of work. Still, being free from a sticky platform like WordPress and the experience of simply doing a `git push` to update a blog was a surprisingly pleasant experience and well worth the effort.

Photo by [Robinson Greig](https://unsplash.com/@robinson?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/photos/HrnAxAUwle8?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
