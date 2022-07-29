---
layout: post
title: Embedding a shell script within an AppleScript
date: '2012-08-26T21:28:26+01:00'
tags:
- applescript
- unix
- shell
- terminal
- python
- django
tumblr_url: https://richbs.org/post/30266351395/embedding-a-shell-script-within-an-applescript
---
[Embedding a shell script within an AppleScript](https://developer.apple.com/library/mac/documentation/applescript/conceptual/applescriptx/Concepts/work_with_as.html#//apple_ref/doc/uid/TP40001568-1152618-BGBEFCAH)  

This is an extract from a Applescript I was using to call a management command in Django. It outputs some HTML into a folder on the user’s Desktop. The variables you see below—`appDir` and `theFolder`—are gathered from dialog boxes earlier in the script.

    …
    set pythonPath to ".:~/Sites/:$PYTHONPATH"
    do shell script "export PYTHONPATH=" & pythonPath & "; python " & appDir & "/manage.py build -o ~/Desktop/" & quoted form of theFolder & " --settings=settings_build"
    …

The helpful takeaway here is the `do shell script` command and the `quoted form of` statement which adds single quotes to user input for safer command line execution.

