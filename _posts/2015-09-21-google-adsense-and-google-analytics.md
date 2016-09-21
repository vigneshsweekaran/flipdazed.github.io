---
layout:            post
title:             "15 Minute Miracle: Google Analytics & AdSense for Jekyll Blogs"
menutitle:         "15 Minute Miracle: Google Analytics & AdSense for Jekyll Blogs"
date:              2016-09-21 06:31:00 +0100
tags:              Google Analytics AdSense Jekyll Blog
category:          Website
author:            am
published:         true
redirect_from:     "/google-adsense-and-google-analytics/"
language:          EN
comments:          true
---

This post will get you up and running with Google Analytics and Google AdSense within 15 minutes. 

Ok, so I told a white lie. You have to wait for an activation email from Google but your own time spent shouldn't be longer than 15 minutes.

The following steps will get you up and running in no time.

# Requirements

 - **Jekyll-Based Blog**. If this step confuses you then you probably don't have one!
 - **Google Account** that you are willing to use for signing up to Google AdSense and Google Analytics
 - **Physical Address** to associate with your account
 - **Phone Number** (mobile or landline)


# Obtaining Google AdSense
Phone Verification/Sign Up  **7 minutes**
Email Validation            **Ongoing**
Post Validation             **TBC**

 - Go to the [Google AdSense website](https://www.google.co.uk/intl/en/adsense/start/)
 - Sign up with your google account.
 - Enter the details required
 - Verify your phone number by SMS or a voice message - this happened within seconds for me
 - Await the email verification (Get Google Analytics while you wait)
 - To Be Continued ...
 
# Obtaining Google Analytics
Completion                  **6 minutes**

 - Go to the [Google Analytics website](https://analytics.google.com/analytics/web/)
 - Sign up with your Google Account
 - You will then be directed to a page with a piece of code similar to below, where I have edited the second parameter (used to be the Tracking ID) of the second `ga()` ready for the next step
 
<pre data-line="7" class="line-numbers language-javascript"><code>&lt;script&gt;
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');
  
  ga('create', '{% raw %}{{site.google_tracking_id}}{% endraw %}', 'auto'); 
  ga('send', 'pageview');
  
&lt;/script&gt;
</code></pre>

 - In my Jekyll website directory I created a new file at `_includes/google_analytics.html` and pasted the code above code in that file.
 - In your `_config.yml` file add the line `google_tracking_id:  YOUR-TRACKING-ID`
 - Add the following in within the `<head></head>` section of each page. See [^1] for more details.

<pre class="line-numbers language-jekyll"><code>{% raw %}{% if site.google_tracking %} {% include google_analytics.html %} {% endif %} {% endraw %}
</code></pre>

You are now set up with Google Analytics and can view the latest data by signing in to the [Google Analytics website](https://analytics.google.com/analytics/web/).

[^1]: The location of your `<head>` will vary on your personal organisation method for your `html`. Most users will have a file `_includes/head.html` which contains the `<head></head>` tag. If this is not you then try looking in `_layouts/default.html`. The basic idea is that it should appear in the `<head>` tag for every page.