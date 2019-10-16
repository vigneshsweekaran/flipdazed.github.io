---
layout: post
title: "04: Installing Python Packages"
menutitle: "04: Installing Python Packages"
date: 2019-10-15 22:32 +0000
tags: Python Tutorial Learning
category: Python Tutorial
author: am
published: true
redirect_from: "/2019-10-15-04-Installing-Python-Packages/"
language: EN
comments: true
---

## Learning Outcomes
{:.no_toc}

 - Be able to install libraries from pypi using `pip`
 - Be able to install libraries requiring C++ compilers using `conda`
 - Understand that machines require precision in commands
 - Be able to locate a proxy in a corporate setting

## Contents
{:.no_toc}

* This will become a table of contents (this text will be scraped).
{:toc}

Run the following

    $ python -c "import requests"

we will see how to fix this issue


# `pip`

`pip` is the default `python` package manager. It is to `python` what `homebrew` is to Mac OS X.

Install a new `PACKAGE` with

    $ python -m pip install PACKAGE

for example

    $ python -m pip install requests --proxy https://path.to.my.proxy:1234 -IU

Here we introduce three very useful flags when in a corporate environment. Almost all corporations will have proxies. You need to specify the proxy to hit the internet programmatically.

To see other flags type

    $ python -m pip install -h


# `conda`
Recall `anaconda` is what we installed `python` with. This is a more sophisticated python package manager.

Some libraries like `scipy` require advanced C++ libraries (LAPACK / BLAS) in the case of `scipy`. These can be **extremely** difficult to install on Windows machines in particular *(last time installing LAPACK/BLAS from scratch took me 2 days reading documentation plus 8 hours of compilation!)*

`conda` comes with prebuilt versions and compilers which mean you get scipy in under a minute

    $ conda install scipy

Downsides are that not all libraries are supported by `conda` and there is less help on Stackoverflow for it.


# Exercises

### 4.1. Supplying arguments to a program

Try and figure out what the `-IU` means and notice that you can stack single letter flags on a single `-`. The combination of `-IU` is usually only used when an installation is broken or doing something weird and won't work.

### 4.2. Debugging bad command

Why does `$ python - m pip -h` not work?

### 4.3. Understanding importance of `$PATH`

Recall that the first command must be an executable program. Why does `$ pip -h` work?

Hint: On Windows powershell type `$ $env:PATH` or on Mac OS X type `$ echo $PATH`.
You can use `ls` to display files that match patterns, for example

```
$ ls "/a/directory/*pip*"
```

use this on the `/bin` directory for anaconda.

### 4.4 Corporate Proxies

If you are on a coporate network find your proxy. In windows follow [this guide](https://superuser.com/a/346376). On Mac OS X follow [this guide](https://askubuntu.com/a/924676)

### Next Topic
{:toc}

coming soon ...