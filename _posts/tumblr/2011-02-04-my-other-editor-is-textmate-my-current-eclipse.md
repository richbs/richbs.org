---
layout: post
title: 'My other Editor is Textmate: my current Eclipse set up for web development.'
date: '2011-02-04T19:28:13+00:00'
tags:
- eclipse
- python
- django
- ide
- programming
- android
tumblr_url: https://richbs.org/post/3108289506/my-other-editor-is-textmate-my-current-eclipse
---
[Eclipse](http://eclipse.org/), the legendary Java-based IDE, is a big beast and probably not designed for a quick Apache conf change. TextWrangler, Textmate or Vim are my choice here . But if I’m going to be spending more than 15 minutes on a task, I will eagerly spin up Eclipse.

To keep Eclipse starting up and working efficiently, it pays to be judicious in the extra modules you install.

Here’s how I build my IDE for [Django](http://djangoproject.com/) and general web development.

## Eclipse for PHP Developers

I start by downloading [Eclipse for PHP developers](http://www.eclipse.org/downloads/packages/eclipse-php-developers/heliossr1).

I prefer programming in Python but as a web developer you’re always likely to be fixing someone else’s PHP at some point.

## Web Tools

This package gives you the [Eclipse Web Tools Platform](http://www.eclipse.org/webtools/) for free, with good, basic content assist and syntax highlighting for HTML, CSS and JavaScript.

## PyDev

The first and most important package I add to Eclipse is [PyDev](http://pydev.org/), giving me everything I need to work with Python. It knows more about the code I’m writing than I do. PyDev content assist is powerful and code style correction makes sure my syntax conforms to [PEP 8](http://www.python.org/dev/peps/pep-0008/).

- PyDev Update Site: [http://pydev.org/updates](http://pydev.org/updates)

## Version Control

[SubClipse](http://subclipse.tigris.org/) for brilliant Subversion integration and EGit for my emerging Git habit. EGit is in incubation and is available from the core Eclipse update site, available from the help menu _Install New Software…_.

- SubClipse (1.6) Update Site: [http://subclipse.tigris.org/update\_1.6.x](http://subclipse.tigris.org/update_1.6.x)

## Other Eclipse perks

### Local History

Even between SVN or Git commits you might lose an hour or two’s work by a careless cut and paste. Eclipse is constantly tracking your code changes and will even let you restore an entire file from local history if you mistakenly obliterate it from the file system. This alone is worth the Eclipse learning curve. Eclipse tracks your history from the moment you start a project, without even giving you the option to make the error of turning it off.

### Terminal (command line)

To lessen the need to switch away from Eclipse when “in-the-zone” I’ve got the built in terminal panel working. It is _incubating_ and it’s not perfect out of the box but a quick `source /etc/profile` and `source ~/.bash_profile` got the environment the way I like it on my Mac.

### Android

The recommended development platform for [Android is Eclipse ADT](http://developer.android.com/guide/developing/eclipse-adt.html) and there are loads of tutorials and videos that assume this combo.

I’m quite excited about playing with Java for mobile within my Eclipse comfort zone.

- ADT Update Site: [https://dl-ssl.google.com/android/eclipse/](https://dl-ssl.google.com/android/eclipse/)
