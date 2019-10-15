---
layout: post
title: "The Python Interpreter"
menutitle: "The Python Interpreter"
date: 2019-10-15 22:32 +0000
tags: Python Tutorial Learning
category: Python Tutorial
author: am
published: true
redirect_from: "/2019-10-15-The-Python-Interpreter/"
language: EN
comments: true
---


## Learning Outcomes
{:.no_toc}

 - Able to launch / exit python in the terminal and run a simple command
 - Able to import libraries
 - Check python version
 - Understand weakness of jit compilation

## Contents
{:.no_toc}

* This will become a table of contents (this text will be scraped).
{:toc}

# Hello World

Open the `Terminal` and type

```sh
$ python
```
This should then load the **basic** python interpreter. When we are in a python interpreter, I will use the following syntax: `>>>` for new logical blocks, `...` for line continuations or logical block continuations and results will be without padding.

Try entering the following to print thing to the console window

```python
>>> 'Hello World'
Hello World
```

This gives the string representation of the string `'Hello World'` which is by definition `Hello World`


# Statements
A statement in `python` is something that is expressed in a single logical line such as

```python
>>> import sys
```

or another example is the `assert` statement which throws an error unless the following expression evaluates to `True`

```python
>>> assert True
```

You may be familiar with some other statements like

  - `break`
  - `continue`
  - `return`
  - `yeild`
  - `raise`

amongst others which can be found on the [docs page](https://docs.python.org/3/reference/simple_stmts.html#grammar-token-expression-stmt)


# Using `python` libraries

To use python libraries we use the statement `import`

Worth noting is the following to check what `python` program is currently running your script (can be very important for debugging strange errors!)

```python
>>> import sys
>>> print(sys.executable)
/usr/local/anaconda3/bin/python
```

and for checking what version of `python` you have running. We should **all** be using at least python 3.6 by now (higher versions are fine)

```python
>>> print(sys.version)
3.6.5 |Anaconda, Inc.| (default, Apr 26 2018, 08:42:37)
[GCC 4.2.1 Compatible Clang 4.0.1 (tags/RELEASE_401/final)]
```

if you get `ModuleNotFoundError` this library is a non-standard `python` library and we can address that in another section.


# Exiting `python`
You can exit `python` by entering

```python
>>> exit()
```

but most usually `Control+D` (I usually jam it a few times to avoid the "Are you sure" etc.)
If you are in the middle of executing code, you can force code to stop by entering `Control+C`. Sometimes you may have to hold `Control+C` or press it quite a few times to actually register particularly if you are doing something where the interpreter hasn't got time to register your command!


# Exercises


### 3.1 The issue with just-in-time (jit) complilation

In the `python` console

```python
>>> HelloWorld
```

Notice that the compiler tells you three things

 - what file the issue was in
 - what line the issue was on
 - an arrow signalling which bit of code was bad
 - An error type and description of the error

Now try

```python
>>> def hello_world():
...     print(HelloWorld)
```

and then

```python
>>> hello_world()
```

this is one of the biggest weaknesses of `python` and why most core quant libraries are still built in `java` or `C++`. Most of the bugs you create will be due to this.

NB: You're not expected to fix this code yet! Just move onto the next exercise :)