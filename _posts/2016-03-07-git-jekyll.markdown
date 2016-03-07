---
layout: post
title: Using jekyll with git
date: 2016-03-07 13:33:00
categories: linux web blog jekyll git
---
##Overview
This is an article to cover setting up jekyll and git to successfully deploy a blog on a VPS.

##Summary
There are several steps needed to successfully setup a server with git and jekyll.  I use
Ubuntu 14.04 LTS as my VPS OS and the latest Jekyll Ruby gem.  Be sure to run these instructions
on both the server and the client

* Install git and ruby
* Install jekyll
* Create new jekyll site
* Init git repo
* Create git bare repo
* Create git hook
* Create virtual host
* Clone git repo
* Make blog post
* Push back to origin

###Install git and ruby
```
apt-get install git ruby
```
###Install jekyll
To install for the local user only run

```
gem install jekyll
```

To install system wide

```
sudo gem install jekyll
```

###Create a new jekyll site
Create a default new jekyll site (will be created in a directory the same name of the site)

```
jekyll new <site-name>
cd <site-name>
```

###Init git repo
```
git init
git commit -a -m "Initial commit"
```

###Create git bare repo
If you don't have a default location for your git repos please create on.  I use $HOME/git

```
cd ..
git clone --bare <site-name> $HOME/git/site-name.git
rm -rf <site-name>
```

### Create git hook
Git hooks are commands to run after an event in git.  We want to create the site after a commit.  This is 
a post receive hook.  

```
cd $HOME/git/site-name.git/hooks
vi post-receive
```

In the post-receive file, paste in the following code

```bash
#!/bin/bash

GIT_REPO=$HOME/git/site-name.git
TMP_GIT_CLONE=$HOME/tmp/webrepo
PUBLIC_WWW=/var/www/site-name

git clone $GIT_REPO $TMP_GIT_CLONE
jekyll b -s $TMP_GIT_CLONE -d $PUBLIC_WWW
rm -Rf $TMP_GIT_CLONE
exit
```

###Create virtual host
This is a bit involved so I will simply provide a link to instructions

[Apache configuration](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts)

###Clone git repo
On your workstation, clone your git repo

```
git clone ssh://username@server-name/home/username/git/site-name.git
```

###Make a blog post
It is easier to edit the default post in site-name/\_posts.  Edit this file and then run the following

```
git commit -a -m "Edited default post"
```

###Push back to origin
When you push changes back to the origin the post\_receive hook runs and creates your site.

```
git push origin
```

After this completes browse to your site.
