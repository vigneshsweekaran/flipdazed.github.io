---
layout: post
title: "Getting to grips with the Terminal"
menutitle: "Getting to grips with the Terminal"
date: 2019-10-15 22:32 +0000
tags: Python Tutorial Learning
category: Python Tutorial
author: am
published: true
redirect_from: "/2019-10-15-Getting-to-grips-with-the-Terminal/"
language: EN
comments: true
---

Learning outcomes

 - Be able to find / open the terminal
 - Be able to navigate filesystem within the Terminal
 - Basic usage of linux commands


# Opening the `Terminal`

Why do I care? Basically because all the self help on the internet will rely on you being able to reproduce what you messed up in a minimal example. A lot of errors at the early stages are often to do with set up and nothing to do with `python`. Being able to run code in a minimal way in the terminal will give you confidence in programming in `python` and be able to isolate `python` issues from mor generic setup or configuration issues which can be very difficult to otherwise debug.

## Opening a console on Mac OS X
OS X’s standard console is a program called `Terminal`. Open `Terminal` by navigating to Applications, then Utilities, then double-click the `Terminal` program. You can also easily search for it in the system search tool in the top right.

The command line `Terminal` is a tool for interacting with your computer. A window will open with a command line prompt message, something like this:

    mycomputer:~ myusername$

## Opening a console on Windows
Window’s console is called the Command Prompt, named `cmd`. However it sucks. I would always recommend using `powershell`. An easy way to `powershell` to it is by using the key combination `Windows+R` (Windows meaning the windows logo button), which should open a Run dialog. Then type `powershell` and hit `Enter` or click Ok. You can also search for it from the start menu. It should look like:

    C:\Users\myusername>

Window’s `powershell` is not quite as powerful as its counterparts on Linux and OS X but they've copied all the relevant parts that we will want to use


# Navigating around the `Terminal`

From this point onwards anything denoted like

    $ program

will represent a `Terminal` interface (this is not python!). The above represents running a command named `program` in the `Terminal`.

## Moving around directories

These commands will soon become second nature & after some practice they are quite a lot faster than navigating the user interface file explorer!

Check what directory you are in

    $ pwd

show what's in the current directory

    $ ls

make a new directory

    $ mkdir newdirectory

enter this directory

    $ cd directory

go up one directory

    $ cd ..

show whats inside the directory

    $ ls directory/

remove a file

    $ rm file.ext

Change the path (rename) a file (and / or extension)

    $ mv directory/file.dat anotherdirectory/file2.csv

remove a directory (I probabaly wouldn't do this hastily!)

    $ rm -r directory

The `-r` is a known as a flag. This modifies the behaviour of the program `rm`. IN this instance it allows `rm` to delete directories which is otherwise a risky default ability.

These are pretty much all the commands you need. Try to familiarise yourself with them.
All of these commands e.g. `ls`, `rm`, `mkdir` are all programs themselves that take in arguments from the command line depending on their operation.


# Exercises

Your *local* machine home directory is always denoted by `~/` (`~\` on Windows)

  1. Create a new directory named `temp` in this directory

  2. Change the name of this directory to `pylrn`

  3. Run the following command to create a new file

        $ touch ~/pylrn/test.py

    verify it is there with

        $ ls ~/pylrn/*.py

    Put something inside it

        $ echo "print('Hello World')" >> ~/pylrn/test.py

