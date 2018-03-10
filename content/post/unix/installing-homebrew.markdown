---
title: Installing Homebrew on OSX Mavericks
date: 2013-12-31
comments: true
categories: [homebrew, osx]
keywords: "homebrew, osx"

aliases: 
  - /blog/2013/12/31/installing-homebrew-on-osx-mavericks/
  - /2013/12/31/installing-homebrew-on-osx-mavericks/
---

I choose to install all my applications via [Homebrew](http://brew.sh).  It's easy to install, easy to update, works well for me and has all the required packages including everything required for Haskell.  Installing Homebrew is as easy as running the following command in terminal[^brew]:

``` bash
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
``` 

<!-- more -->

You will be required to enter your administrator password and might be asked to install Apples Xcode developer tools and also Git versioning tool.  This is easy in Mavericks as the UI will prompt you for any requirements.

{{< figure src="/images/unix/install-xcode.png" >}}
{{< figure src="/images/unix/install-git.png" >}}
{{< figure src="/images/unix/download-homebrew.png" >}}
{{< figure src="/images/unix/install-homebrew.png" >}}
{{< figure src="/images/unix/homebrew-installed.png" >}}

After all the requirements are met and everything is downloaded and installed you will be requested to run `brew doctor` to complete the installation, you should see something like this:

``` bash
==> Installation successful!
You should run `brew doctor' *before* you install anything.
Now type: brew help
$ brew doctor
Your system is ready to brew.
``` 

For more information visit the Homebrew wiki: [http://github.com/Homebrew/homebrew/wiki](http://github.com/Homebrew/homebrew/wiki)
