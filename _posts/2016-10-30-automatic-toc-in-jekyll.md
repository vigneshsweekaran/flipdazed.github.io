---
layout:            post
title:             "Automatic Table of Contents for Jekyll Blogs"
menutitle:         "Automatic Table of Contents for Jekyll Blogs"
date:              2016-10-30 16:58:00 +0000
tags:              Automatic Table of Contents Jekyll Blogs
category:          Website
author:            am
published:         true
redirect_from:     "/2016-10-30-automatic-toc-in-jekyll/"
language:          EN
comments:          true
---

# Contents
{:.no_toc}

* This will become a table of contents (this text will be scraped).
{:toc}

# Introduction
This is more of a personal bookmark so I don't forget this super simple solution

# Method
Include the following in the `_config.yml` file

    markdown: kramdown

then add this line where you want the TOC to appear in the markdown blog entry

    * This will become a table of contents (this text will be scraped).
    {:toc}

This will also include `Contents` inside TOC which looks dumb.

Correct this by `{:.no_toc}` following a title you don't want in the TOC. 

The code for this page up to here is as follows

<pre class="line-numbers language-bash"><code># Contents
{:.no_toc}

* This will become a table of contents (this text will be scraped).
{:toc}

# Introduction
This is more of a personal bookmark so I don't forget this super simple solution

# Method
Include the following in the `_config.yml` file

    markdown: kramdown

then add this line where you want the TOC to appear in the markdown blog entry

    * This will become a table of contents (this text will be scraped).
    {:toc}

This will also include `Contents` inside TOC which looks dumb.

Correct this by `{:.no_toc}` following a title you don't want in the TOC. 

The code for this page up to here is as follows</code></pre>