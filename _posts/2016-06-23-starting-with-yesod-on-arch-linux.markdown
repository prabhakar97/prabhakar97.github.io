---
layout: post
title: "Starting with Yesod on Arch Linux"
date: 2016-06-23 19:59:39 -0400
comments: true
tags: [haskell, web-apps]
---
As part of my eagerness to learn Haskell, I thought it might be a good idea to start building real world applications in Haskell. I chose
[Yesod](http://www.yesodweb.com/) to start building a dynamic web application backed by as database. However, the getting started page on
Yesod website didn't exactly work as advertised, probably because the [Haskell Stack](http://docs.haskellstack.org/en/stable/README/)
had some API change. These steps worked for me.

* Install haskell-stack. It lets you develop, build and test Haskell apps without creating dependency version issues across projects.
If you are coming from a ruby world, it is similar to rbenv.


`sudo pacman -S haskell-stack`


* Checkout the *yesod-mongodb* template. This is going to create a `haskell-webapp` directory and checkout stuff from the `yesod-mongo` template.
Make sure that you don't provide a directory name which is the name of a package, like yesod, ghci etc.

`stack new haskell-webapp yesod-mongo`

* Install [libtinfo from AUR](https://aur.archlinux.org/packages/libtinfo/) because Arch screwed up (from stack perspective). There is
another nuance to take care of. Edit the *PKGBUILD* and ensure that the line that simlinks `/usr/lib/libtinfo.so.5` is uncommented. Otherwise
the next step doesn't work.

* Setup GHCi and friends.

`stack build yesod-bin cabal-install --install-ghc`

* Build the libs.

`stack build`

* Hello World!

`stack exec -- yesod devel`

To access your server visit [http://localhost:3000](http://localhost:3000)
