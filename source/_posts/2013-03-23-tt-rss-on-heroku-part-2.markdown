---
layout: post
title: "TT-RSS on Heroku Part 2"
date: 2013-03-23 17:08
comments: true
categories: 
---

This is my follow-up article to [Replacing Google Reader with TT-RSS on Heroku](http://projectdelphai.github.com/blog/2013/03/15/replacing-google-reader-with-tt-rss-on-heroku/). That article was just my quick notes on how I had set up tt-rss on Heroku as a quick transition from Google Reader. As with any project, nothing is ever perfect the first time and there were a lot of updates that I did until I was comfortable with the setup. Besides some small changes and bug fixes, the biggest two events would be the creation of a installation script and the addition of a self-updater. If I come across anything else, I'll either post it at the end of this article or write a Part 3. I'm still debating.

<!-- Read more -->

Cost
--------
I was indirectly alerted by Survector in [his comment](http://projectdelphai.github.com/blog/2013/03/15/replacing-google-reader-with-tt-rss-on-heroku/#comment-834371937) that I might not have fully explained what Heroku was, its pricing, and its pros and cons. Heroku is a [PAAS](https://en.wikipedia.org/wiki/Platform_as_a_service) (Platform As A Service) which allows developers to host their applications under free or premium accounts. Primarily used for Ruby on Rails projects, Heroku has support for many languages (and unofficial support for PHP which was used for ttrss). Each time a Heroku application is created, the developer is basically given a small computer that has 512 mb RAM running Ubuntu on which the application is run. This computer/server is called a dyno and it operates on a sequential basis meaning that it can only only task at a time. In my ttrss server, I created a Heroku application and gave one dyno the task of running the tt-rss server. Because I was only using one dyno, I was able to stay on the free tier: there were NO costs to run this server. This was nice, because the hosting was taken care of with no costs. Because it the server was only used by me, the dyno could handle the load and I had no need to upgrade. 

Emulating Google Reader
-----------
Thanks to [#6 on Jan's post](http://brasserie-seul.com/?Recipes&nr=50), I was able to emulate the posts as it appeared on Google Reader where by clicking on an article, it would expand to show the feed article. This combined expandable view was much better than the default split view. This can be enabled in the preferences section. I have also started changing the css with my own custom css to have bigger text and more whitespace. There are plugins/patches on the tt-rss forum if you want to use a shortcut or you can use the customize button in the preferences. I first used the live css editor (available in chrome or firefox) to change up the layout and then copy and pasted it into the customize css option to save them. If you want a tutorial, ask in the comments and I'll eventually provide one.

Shell Script
-----------
Inspired by a [post](http://tt-rss.org/forum/viewtopic.php?f=16&t=1360)  on the tt-rss forum in which who_me created a script that allows easy installation of tt-rss on openshift, I decided to make it easier to create tt-rss servers on Heroku. [Hosted on github](https://github.com/projectdelphai/ttrss-on-heroku), the script is pretty stable (hopefully). It's just a bash script that runs through all the commands and should successfully create a server for you. I'll occasionally update it with the correct version number. It sets up the server and a self-updating feature which will be explained later on.

New Versions
----------
Right now, there is no good way to update to the newest version. I'm thinkin of enabling the plugin so that one can turn it on in the preferences. I haven't tried to update to 1.7.5 yet, so I will post if something goes wrong. Until further notice, 1.7.4 will by default be uploaded.

Now to more code and file related updates. . .

Procfile and web-boot.sh (bug)
-----------
For some reason, the default httpd.conf for PHP files has a MaxClients of 1 meaning that only one person can connect to the server. Obviously, this can be a problem. So I had to change it using a custom worker. Workers are another name for the task that is assigned to a dyno. So I created a file called Procfile in the top directory and in it inserted this line

	web: sh www/web-boot.sh

This tells the Heroku dyno to run the web-boot.sh file instead of the default boot.sh file on Heroku's root server. So obviously, the next step would be to create a web-boot.sh script. In the same directory as the Procfile, create a web-boot.sh file and put these lines in it:

	sed -i 's/^ServerLimit 1/ServerLimit 8/' /app/apache/conf/httpd.conf
	sed -i 's/^MaxClients 1/MaxClients 8/' /app/apache/conf/httpd.conf

	sh boot.sh

This tells Heroku to up the ServerLimit to 8 as well as the MaxClients. After that, run the default boot.sh file to start tt-rss. Pretty easy so far.

Self-Updating
---------
This definitely was my greatest accomplishment so far. The problem here is that tt-rss doesn't have a real-time updating solution like google reader did. This is basically because constantly polling for updates would take up a lot of resources that a self-hoster doesn't want to do. And so tt-rss has a update.php script that can be run to update all feeds. tt-rss doesn't have a built in update-all button, because if someone has a lot of feeds, it would take too long. For example, I have a little over 50 feeds and just importing them and updating them initially all took ~10 mins. Right now, after syncing it all, my feeds update at about a rate of 1/2 a second per article. This can be a problem to update over http because most servers (like Heroku) have a 30 second timeout. So while Jan's post (mentioned above), does mention a url to update, Heroku times out and crashes the application. And when it crashes, it stays crashed for at least 5-10 mins-which is unacceptable. Previously, I had a cron job run every 5 mins to update the feeds, but if my computer was off, nothing would update and if I'm gone from home, I have no way to update everything. 

But I couldn't get Heroku to update the tt-rss server either. Heroku only allows one dyno per application on the free tier. If I had a worker dyno (background jobs) to my web dyno (application job), it would bump up the price to $34 per month which was way to much. And so I threw every idea I had at it, but nothing would work. The one idea I had that seemed the most profitable was to create another Heroku application that could somehow interface with my tt-rss server and remotely update it, but I had no idea how to do this.

After a couple of days of wrestling with it, I came across [this post](http://ar.zu.my/use-two-dynos-on-heroku-for-free/) by arzumy which fully solved my problem. Basically, you have to create two applications in the same folder so that each folder. First create the first application like how we did in Part 1. Remember to use the Procfile and the web-boot.sh file this time instead. However, before you commit the changes to Heroku, add to the Procfile this line:
	
	worker: while true; do ./php/bin/php -c www/php.ini ./www/update.php -feeds; sleep 300; done

This adds a worker dyno which will run the update script every 5 mins. But someone smart is yelling at me, "Hey! You can only have one dyno!" And that person would be right. So here's how to get around it. Push what you have to your heroku application so that works. Go to your tt-rss server online and make sure it works. If it does, we're ready to move on.

In the same folder as your application, create a new application like this

	heroku apps:create \<appname\>-updater

Now you need to connect your new application to your old one. Forewarning, because you now have two applications created in the same folder, you always need to specify which app you're talking about with "--app \<appname\>. Grab the database url for the tt-rss server with

	heroku config --app \<appname\>

Copy and paste the database url into your new updater application as well as adding the proper environment variables.

	heroku config:add DATABASE_URL=\<databaseurl\> --app \<appname\>-updater
	heroku config:add LD_LIBRARY_PATH=/app/php/ext:/app/apache/lib --app \<appname\>-updater

Now add the application repository to your git config with

	git remote add \<appname\>-updater git@heroku.com:\<appname\>-updater.git

Now push your data to your second application with

	git add .
	git commit -m "initial commit"
	git push \<appname\>-updater master

You might also want to push all your data to your ttrss server just in case with the same command as above just replacing the \<appname\>-updater with heroku.

Now you need to assign your workers. Your first application will still only be using the web dyno as it was originally. However, Heroku has this ability where you can have one worker dyno instead of one web dyno and it is still totally free. So assign a worker dyno to your heroku config with

	heroku ps:scale worker=1 --app \<appname\>-updater

If everything worked out well (and I didn't forget anything), you should now recieve updates every 5 minutes. You can check this by running
	
	heroku logs --app \<appname\>-updater

Now, I have a tt-rss server that self-updates and provides rss feeds for me whenever I need them. Besides, this version updating (which is technically not even necessary), I can consider this project sufficiently completed. 

I'm pretty sure that I've cited all my external sources in the text above, but nevertheless: thanks to all who provided me with information and tutorials especially arzumy who in a couple of paragraphs solved all my updating woes. 
