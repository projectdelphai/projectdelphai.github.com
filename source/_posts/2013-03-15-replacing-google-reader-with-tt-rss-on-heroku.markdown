---
layout: post
title: "Replacing Google Reader with TT-RSS on Heroku"
date: 2013-03-15 14:25
comments: true
categories: 
---

As many probably know, Google Reader is slated to be [shut down](http://googlereader.blogspot.com/2013/03/powering-down-google-reader.html) on July 1, 2013. I use Google Reader all the time and this came as a huge shock to me. My first instinct was to switch to something else such as [Feedly](http://www.feedly.com/). The problem is that I also use [newsbeuter](http://www.newsbeuter.org/) as a desktop client which syncs with Google Reader. It only syncs with Google Reader, Bloglines (now dropped), and [Tiny Tiny RSS](http://tt-rss.org/redmine/projects/tt-rss/wiki). So naturally, I looked up Tiny Tiny RSS to see if I could migrate there. Unfortunately, ttrss (Tiny Tiny RSS) is supposed to be hosted on a server and I didn't want to host it on my computer just for rss feeds. So I decided to host it on [Heroku](https://www.heroku.com/).

<!-- more -->

There were three routes I could have gone. I could have dropped newsbeuter and gone to another client, but I didn't want to do that, because, frankly, newsbeuter is the mutt of rss readers. I also could have just worked with an online-only service, but again, I love newsbeuter. I see no need to drop it, because of Google's choices. The last and what seemed to be the most viable was to use newsbeuter by itself with no online features. However, if I'm bored and at another computer (Windows, Linux, or Mac), I want to be able to read my feeds. 

So I looked up ttrss. ttrss was an exceptional service because it was an application rather than a web-service. I've read many article that show the best Google Reader alternatives yet they were all web-based. The first thing I thought was, "What if they decide to shut down like Google Reader?" ttrss is unique in that it is hosted by the user. I would host the application myself and always have the data. Even if ttrss shut down and support was dropped, I would still have the application running by itself. And it was open-source so somehow it would always live on.

So I had to figure out a way to make ttrss work. I could always host it on my computer or even my [raspberry pi](http://www.raspberrypi.org/), but it seemed like an unnecessary waste of resources. I wanted it to be as simple as Google Reader was. I didn't want to have to have my computer on all the time nor my internet connection especially if I was on a vacation or away from my computer. So I turned to online hosting - free online hosting that is.

I had dabbled with Heroku before with [Ruby on Rails](http://rubyonrails.org/), so I knew that it would work fine for my needs. While Heroku was primarily for Ruby on Rails applications, I remember reading that it had support for PHP which is what ttrss was written in. Here's the process by which I managed to do it.

First, sign up for a heroku account and download the [heroku toolbelt](https://toolbelt.heroku.com/) (if you don't have [git](http://git-scm.com/) installed as well, install it). You'll also need to install [postgresql](http://www.postgresql.org/) on your system so that you can set up the database for Heroku. You'll also need to authenticate your heroku to allow it to upload and create applications. 

	heroku login

and use your Heroku credentials. Create a folder that you will use to hold all your files. I named it like this \<username\>-ttrss (so for me wmcscrooge-ttrss), just to create a standard. Throughout this article, if you chose a different name for your application, just replace \<username\>-ttrss with your application name. In this folder, type

	git init
	heroku create \<username\>-ttrss

The first command creates your repository of code that'll hold all your data. Git is a great tool that manages your version history and I would strongly recommend looking into it if you don't use it already. The second command create a heroku application and tells git where to upload the data to. From here, you'll need to download the tarball from the ttrss website which holds all the application files. Extract all the files (tar -xvzf) and copy all the files inside the Tiny-Tiny-Rss-1.7.4 to the root directory of your \<username\>-ttrss folder.

Now you need to create the database that Heroku/ttrss will use to hold your rss feed information. If you run 

	heroku pg:info

inside your folder, you'll notice that there is no database right now. So the first thing is to add one

	heroku addons:add heroku-postgresql:dev

You could always use a better database, but the basic dev plan is free and should be sufficient at first. ttrss also supports mysql databases, but postgres databases are preferred and you won't really notice a difference if you have more experience in mysql (like I did). Now we need to look up some information on your databases. If you go to Heroku's website, log in, and look at your applications, you will see your application logged there. You can open it if you want, but right now, there's nothing there to see since you haven't uploaded your application yet. All you'll see is a welcome sign if you try and open your application. It is a good thing to check, however, that you do have a postgres database logged in the addons section, that you can open the application welcome page, and that you're operating a cedar stack (which you should be by default). 

The next thing is to move to the [postgres heroku page](https://postgres.heroku.com/) to view your database information. You can login using your Heroku credentials. From there switch to the databases page and click on the database that matches with your site. It will have the format of \<username\>-ttrss::\<databasenickname\>. On your database page, record the Host name, the Database name, the User, and the Password (You'll need to click show).
   
Move back to your application folder, and create a copy of the config.php-dist file and name it config.php. Open up the config.php file in your editor and change the values of DB_HOST to your Host name, DB_USER to your User name, DB_PASS to your Password, and uncomment the DB_PORT section and keep it as 5432. You can keep the DB_TYPE as pgsql since your using postgres instead of mysql. The last thing you'll need to change here is the SELF_URL_PATH to 

	http://\<username\>-ttrss.heroku.com/

Now you're almost done. Move into the schema folder. The thing is that your heroku application has a blank database with no order or form to it. Instead of creating it by hand, ttrss has a schema file that can tell Heroku what the database should look like and you just need to import it. You'll need to connect to Heroku's remote database using postgres's psql command. There are two ways to do it. The first way is the way I did it and the second is a way that I found out later. I haven't used the second way but it seems easier (in that it's not so much copy and pasting), but either way should work.

	psql -h \<Host name\> -U \<User name\> \<Database name\>

Your password will be your Database password. Then import the schema file with

	\i ttrss_schema_pgsql.sql

You will see a lot of text scroll by and once it's finished you can exit with \q. Now you could upload the data and you should be done. If you do it however, you'll find out that the application doesn't run. A quick look through 

	heroku logs

will reveal this error:

	PHP Fatal error:  Call to undefined function mb_internal_encoding() in /app/www/include/functions.php on line 19

Basically PHP doesn't come with mbstring enabled and you need to compile it and install it for Heroku yourself. Thankfully its been already done for us. In a separate folder, download the necessary files

	git clone https://github.com/yandod/heroku-libraries.git	

Move into heroku-libraries/php/mbstring and copy both files (mbstring.so and example-php.ini) into your \<username\>-ttrss root folder while renaming the example-php.ini file to php.ini.

Now you should be able to finish up. Go to the root folder (should be there anyway) and enter these commands to clean up your git repo and upload it to heroku.

	git add .
	git commit -m 'Initial commit'
	git push heroku master

Because I've used git before, I was able to push the files across easily. If it's your first time, you may have a bit of trouble and I won't be able to help you here as I cannot emulate a fresh install. A quick google search should help you if you run into any problems. Open your application with

	heroku open

which will launch it in your default browser or open it yourself with \<username\>-ttrss.heroku.com. You should see a login screen come up. Log in with the default user:admin and pass:password. Change your password right after logging in. From here, you can explore the settings and manually add feeds as you want. Of course, since you're migrating from Google Reader you'll want to be able to import your feeds. In Google Reader you can export your data using Google Takeout. Once you've downloaded your data, extract the files and pull out the subscriptions.xml file. Rename it to subscriptions.opml and then import it using preferences>Feeds>OPML. Now you have a working Google Reader alternative where you have full control over everything.

 The only issue is that everything doesn't automatically update. If you want to update your feeds, you either have to do it manually per feed, use SIMPLE_UPDATE_MODE, or write a script to update your feeds. In your config.php you can change SIMPLE_UPDATE_MODE to true and upload the changed config to Heroku which should slowly update your feeds one by one on its own, however, this will only work if you have ttrss open in your browser. I did not try this method and have no input on how well it works. Instead I chose to write a script. Heroku has a function using 

	heroku run bash

where you basically have a sort of ssh feature into your data in your application. Using this you can run the update.php file every x minutes. Basically I created a bash script that says

	#! /bin/sh
	cd /home/<user>/path/to/ttrss/folder
	heroku run './php/bin/php -c www/php.ini ./www/update.php -feeds'

This tells heroku to run an update (./www.update.php) using php (./php/bin/php) with mbstring enabled (-c www/php.ini). I placed this script in /usr/local/bin after 

	chmod +x \<filename\>

and ran it to see if it worked. After it successfully ran, I create a cron script so that it would run every 30 minutes.

	crontab -e

which opens up the cron file and then added in 

	*/30 * * * * /usr/local/bin/\<filename\>

You can change the 30 minutes to whatever you want though I would not make it any less especially if you're low on resources. I am still figuring out if I can get Heroku to run this script without my computer, but that's a work in progress.

From here I can access ttrss from any browser, from my computer (using newsbeuter which I won't document here; use the git version if you're going to use it), and from a phone (I believe there's an android app, not sure though). 

I realize that the faulty part of this method is Heroku as at any point, Heroku could shut down as well, but that's not the point. There are multiple other free hosting sites that I could have used. I looked at appfog.com for instance and [this page](http://www.quora.com/Is-there-anything-like-Heroku-I-can-use-for-a-PHP-site) has a ton of other resources as well. It may even be possible to get it running on github pages. But ttrss is built so that as long as I have a place to host it, I can easily migrate the installation and that's what makes it great.
