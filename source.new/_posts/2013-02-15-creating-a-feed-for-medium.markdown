---
layout: post
title: "Creating a Feed for Medium"
date: 2013-02-15 15:47
comments: true
categories: 
---

More and more often, I've found myself reading posts from [Medium](http://www.medium.com). It is a relatively new site (still in closed beta for writers) that was created to be a "place for ideas". While at first, it may seem a lot like sites such as reddit or Hacker News, it's a lot different as well. It's different in that it feels like a magazine. I enjoy reading the articles on Medium and it looks beautiful. From what I've heard, writing in Medium is great as well, to the point that one writer said that he wrote emails in it just because of how much he enjoyed the UI. It also deviates from just technology articles. While it has those, it also contains articles that just express ideas or thoughts that have occurred to people. No matter the content, however, the writing is how all posts should be written: easy-to-read, thought out, and creative. 

<!-- more -->

So after realizing how much I enjoyed reading articles from Medium, I looked for the rss feed. I hate going to sites every day to check on new posts and so Google Reader and newsbeuter have become essential. And I found what I believe is one of Medium's largest flaws: it doesn't have one collective feed. Instead it has multiple feeds for its collections (think tags). So I was faced with the dilemma of what to do; I wasn't willing to hear about great article through 3rd parties, yet I would not remember and refused to check every day. I turned to [Feed43](http://www.feed43.com). 

I'm sure there are better options out there, but Feed43 has helped me when I need quick solutions and I haven't invested the time needed to find a better one. Here's a guide written as to how I created a basic feed so that others can learn how and so that I can remember how when I inevitably need to make more feeds.

NOTE: Feed43 does have a limit to the number of queries it can make along with other limitations. However, until further notice, it is the best and easiest service for me. 

I like Feed43, because I can create the feed myself based on a site's HTML source rather than relying on an automatic service. After signing in, I created a new feed and agreed to the terms of service. Then, I filled in which site I wanted to create a feed from and viewed the html source as seen below. <br>

![Step 1](/assets/images/Medium_Feed/Step_1.jpg)

I've always used % for the global search pattern to the point that I don't even know if something else is accepted. There are probably more options and limitations, but for my purposes, % has always worked. The key to the Item Search Pattern is to find the essential html code that corresponds to the posts on the main page of your site. Here is where you will need to know some basic HTML. I would suggest going out and learned at least some basics before you try this. I believe that you could puzzle it out with some logic, but you can get pretty lost finding out where you need to be. So I knew the first post title was called "I'm a Mac. You're a PC. . .". So I scrolled down until I found that part in the extracted source. By comparing the code between post titles, I noticed that the post title would appear shortly after 
	
	<h3 class="post-item-title">

After that, I isolated the article link to the href after the h3 header, the title to the title tag, and the brief summary to the paragraph tag shortly after that. From there I copied that bit of code into the Items Search Pattern box replacing the parts where the important information was with %. Any bits that were not essential to identifying where in the code I was, I replaced with a *. This means that there's a lot of code there, but feed43 doesn't have to pay attention to that and can skip it until the next important part. This is what the final product looked like.

![Step 1](/assets/images/Medium_Feed/Step_2.jpg)

From there, it was a simple matter of just filling in the fields, with the necessary information and % numbers. The hardest part is definitely Step 2. Here you you can see how I finished up. I previewed the rss to make sure it worked out fine, renamed the feed, and then added it to google reader. 


![Step 1](/assets/images/Medium_Feed/Step_3.jpg)

![Step 1](/assets/images/Medium_Feed/Step_4.jpg)

From here, I've been thinking of where to go. While this method works, it isn't very helpful to deploy. I can't exactly share the feed, because I doubt that many people can use the same feed without it shutting down. Another option I've been thinking of is to use github or heroku to create the xml file and spew the items back whenever google reader queries for new information. I know ruby has a way of creating rss feeds so it might be possible with heroku. Sooner or later, I'll see into creating a website that can store multiple, custom rss links. It may work, it may not.
