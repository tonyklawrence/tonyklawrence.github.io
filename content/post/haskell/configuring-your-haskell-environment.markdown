---
title: Configuring your Haskell environment
slug: configure
date: 2014-01-01
comments: true
categories: [haskell, functional_programming]
keywords: "haskell, functional programming, sublime text"

aliases: 
  - /blog/2014/01/01/configuring-your-haskell-environment/
  - /2014/01/01/configuring-your-haskell-environment/
---

My love of functional programming has been getting stronger over the past year so I decided to attend [Well-Typed](http://www.well-typed.com/) Haskell courses at Skills Matters[^wt].  As I'm a huge fan of JetBrains [IntelliJ](http://www.jetbrains.com/idea/) IDE I found using Haskell a little lacking in this area (unless you can be online with [FP Complete](http://www.fpcomplete.com/).)

In this article I will explain how I configured my Haskell development environment using Sublime Text 3 and a few extras.

<!-- more -->

## Installing Haskell (via Homebrew[^brew])

If you need to, take a look at my post on how to [Install Homebrew](/2013/12/31/installing-homebrew-on-osx-mavericks/).  It's not actually very hard at all.

Once you have Homebrew up and running it's very simple to install Haskell (although it might take a while to complete.)

``` bash
$ brew install haskell-platform
==> Installing dependencies for haskell-platform: apple-gcc42,,ghc
==> Installing haskell-platform dependency: apple-gcc42
==> Downloading https://downloads.sf.net/project/machomebrew/Bottles/apple-gcc42
######################################################################## 100.0%
==> Pouring apple-gcc42-4.2.1-5666.3.mavericks.bottle.2.tar.gz
==> Summary
  /usr/local/Cellar/apple-gcc42/4.2.1-5666.3: 104 files, 75M
==> Installing haskell-platform dependency: ghc
==> Downloading https://downloads.sf.net/project/machomebrew/Bottles/ghc-7.6.3.m
######################################################################## 100.0%
==> Pouring ghc-7.6.3.mavericks.bottle.2.tar.gz
==> Caveats
This brew is for GHC only; you might also be interested in haskell-platform.
==> Summary
  /usr/local/Cellar/ghc/7.6.3: 5286 files, 776M
==> Installing haskell-platform
==> Downloading http://lambda.haskell.org/platform/download/2013.2.0.0/haskell-p
######################################################################## 100.0%
==> ./configure --prefix=/usr/local/Cellar/haskell-platform/2013.2.0.0
==> make install
==> Caveats
Run `cabal update` to initialize the package list.

If you are replacing a previous version of haskell-platform, you may want 
to unregister packages belonging to the old version. You can find broken
packages using:
  ghc-pkg check --simple-output
You can uninstall them using:
  ghc-pkg check --simple-output | xargs -n 1 ghc-pkg unregister --force
==> Summary
  /usr/local/Cellar/haskell-platform/2013.2.0.0: 1463 files, 232M, built in 24.5 minutes
```

As requested I also run `cabal update` to initialise the package list.

``` bash
$ cabal update
Config file path source is default config file.
Config file /Users/tony/.cabal/config not found.
Writing default configuration to /Users/tony/.cabal/config
Downloading the latest package list from hackage.haskell.org
```

Then check that it's all working correctly.

``` bash
$ ghci
GHCi, version 7.6.3: http://www.haskell.org/ghc/  :? for help
Loading package ghc-prim ... linking ... done.
Loading package integer-gmp ... linking ... done.
Loading package base ... linking ... done.
Prelude> 2+2
4
Prelude> :t 4
4 :: Num a => a
Prelude> 
Leaving GHCi.
```

## Installing Sublime Text 3

Sublime Text 3 is currently still in beta.  I have been using it for a while and have not come across any problems which is why I can highly recommend giving it a go.  It can be downloaded from [http://www.sublimetext.com/3](http://www.sublimetext.com/3).  Once downloaded, open the DMG file and copy to your Applications folder.  On starting you will be greeted with the following:

{{< figure src="/images/haskell/sublime-text-3.png" >}}

If you like sublime please purchase a license to allow the developers to keep supporting this great application[^buy].

## Configuring Sublime Text for Haskell

To allow us to fully use the power of Sublime Text we need to install Package Control.  This will allow us to use the Haskell package later.  Full instructions on installation can be found on the [Package Control Site](http://sublime.wbond.net/installation).  A quick overview follows.

### Installing the Package Control

Whilst in Sublime Text type ``ctrl+` `` to open up the console and paste the following:

``` python
import urllib.request,os; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())
```

The output in the console will probably look something like this:

``` bash
reloading plugin Package Control.Package Control
found 2 files for base name Main.sublime-menu
Package Control: No updated packages
```

If you don't want to trust the script you can follow the manual instructions on the package control website.

### Installing SublimeHaskell Package

SublimeHaskell[^sublimehaskell] is a github hosted project which adds support for many things.  I mainly use it to auto-compile the edited file on save and easily running a Haskell program from the editor.  Full instructions are available at the [SublimeHaskell](http://github.com/SublimeHaskell/SublimeHaskell) project site.  Below I will show how I got everything working.

There are a few packages that need to be installed for this package to work.  Using `cabal` makes this easy (although it might take a while.

``` bash
$ cabal install aeson haskell-src-exts haddock hdevtools
```

Now, to install the package in Sublime Text 3 you can type `cmd-shift-P` to open up the search window.  Type in `install` to locate the install package option and select it.  You should see  in the status bar that it is going to the repository to find available packages.

{{< figure src="/images/haskell/install-package.png" >}}

Next type in `haskell` to locate the SublimeHaskell package and select it.

{{< figure src="/images/haskell/install-sublimehaskell.png" >}}

The status bar should indicate that it is installing the package and when it is completed.  Lastly we need to configure the package to use hdevtools as this isn't the default behaviour.

From the menu choose `Preferences` -> `Package Settings` -> `SublimeHaskell` -> `Settings - User` to open up the user settings.  Now paste the following into that file and save making sure you replace `<username>` with your username.

``` json
{
  "enable_ghc_mod": false,
  "enable_hdevtools": true,
  "add_to_PATH": [ "/Users/<username>/.cabal/bin" ]
}
```

A restart is then required.

You can now enter your Haskell code into Sublime Text, make sure you have the Haskell build system selected and then type `cmd-B` to save, build and execute the code!

{{< figure src="/images/haskell/haskell-program.png" >}}

[^wt]: [Well-Typed Advanced Haskell by Andres Löh](http://skillsmatter.com/course/scala/well-typed-advanced-haskell)
[^brew]: [Homebrew](http://brew.sh)
[^buy]: [Support Sublime Text](https://www.sublimetext.com/buy)
[^sublimehaskell]: [SublimeHaskell GitHub hosted project](http://github.com/SublimeHaskell/SublimeHaskell)
