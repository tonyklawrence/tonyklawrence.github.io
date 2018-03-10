---
title: Setting up Octopress
date: 2012-08-01
comments: true
categories: [octopress, osx]
keywords: "octopress, osx, mountain lion"

aliases: 
  - /blog/2012/08/01/setting-up-octopress-on-osx/
  - /2012/08/01/setting-up-octopress-on-osx/
---

When upgrading to Mountain Lion I decided to replace my existing Wordpress site with a static one.  There were many reasons for this.  With Wordpress I was unable to easily version control my posts into GitHub.  I also had no control over and backup strategies.  One of my colleagues - [Toby Weston](http://baddotrobot.com) - was running Octopress so I thought I'd take a look.

<!-- more -->

Octopress is based upon Jekyll, developed and used by GitHub.com.  It uses Ruby to generate a static site, this can them be published to a multiple of hosting solutions, I have decided to use GitHub pages.  It looks elegant and can be customised using themes or changing the styles yourself (if you are familiar with html/sass.)  There are also many available plugins to extend the functionality of the site.

Unfortunately, even the latest, greatest OSX 10.8 does not come with all the tools Octopress requires.
 
#### Requirements not met by Mountain Lion

* Ruby version is out of date, requires >= 1.9.3
* Ruby Bundler is missing

You'd think it would be very easy to upgrade Ruby, I found out that this was not the case.  The easiest way is to use a tool named RVM (ruby version manager) but this is only currently available in source for which needs compiling.  Don't worry, this is quite trivial once you have the tools at hand.

#### Tools required by RVM

* Some sort of C compiler (if you have Xcode, you'll be ok)

If like me you like to keep your mac clutter free you'll never be happy installing the mammoth Xcode.  There is a solution, Apple have separated out their developer tools into a small packages that is quick to download and install.

### Installation

So, let's get on with the setup.  First we can install those command line tools from: [Apple Developer Tools](https://developer.apple.com/downloads/index.action)

Next we install the Ruby Version Manager:

``` bash
$ bash -s stable < <(curl -s https://raw.github.com/wayneeseguin/rvm/master/binscripts/rvm-installer)
```

And use it to update Ruby.  I'm going to update to the latest version available at the time of writing, this is 1.9.3.

> to start using RVM in your current terminal session you need to run:
>
> `source ~/.rvm/scripts/rvm`

``` bash
$ rvm install 1.9.3
```
	
> if you choose version <= 1.9.2 you must specify --with-gcc=clang otherwise it will not install.

### Setup

Ok, that's our Mac up to date, now it's time to look at Octopress.

Octopress lives on GitHub so it's really easy for us to pull our own version and keep up with the latest enhancements.  Decide where you want it to live then clone yourself a copy.

``` bash
$ git clone git://github.com/imathis/octopress.git <dir>
$ cd <dir>
```

> you'll be asked if you trust the .rvmrc file, all this will do is switch to the required version of Ruby.  If you didn't install ruby 1.9.3 you will need to update the .rvmrc to require your version to prevent any warnings.

``` bash
$ ruby --version # should report your Ruby version >= 1.9.3
```

Once we've installed the required bundler we are ready to go.

``` bash
$ gem install bundler
$ bundle install
```

Install the default Octopress theme

``` bash
$ rake install
```

This will create 2 directories, one for sass and another for source.  We should now commit these into out repository.

``` bash
$ git add sass source
$ git commit -m "Added initial theme"
```

### Rewards

And now we can run up a local version to test the install.  Run the following command and head to [http://localhost:4000](http://localhost:4000) to see your new Octopress site.

``` bash
$ rake preview
```

You now have a full install of Octopress and can start blogging.  I'd advise you to head over to their deployment site for options and guides on how to publish.
