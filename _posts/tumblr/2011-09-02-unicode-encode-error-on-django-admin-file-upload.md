---
layout: post
title: 'Unicode Encode Error on Django Admin File Upload '
date: '2011-09-02T13:30:00+01:00'
tags:
- django
- linux
- unicode
- suse
- sles
- python
tumblr_url: https://richbs.org/post/9703596496/unicode-encode-error-on-django-admin-file-upload
---
I love tracking down a good UnicodeEncodeError through a web app or any Python code for that matter. I thought I’d share the solution to one of the more low-level exceptions I’d triggered.

Essentially, the test is to upload an image to the Django Admin with a special character in it. My example was this:

bib\_fortüna.jpg

If you can upload an image with this filename and it works, you’re golden. If you get a UnicodeEncodeError, you might have the same issue we had.

    UnicodeEncodeError at /admin/cal/event/add/
    ('ascii', u'/srv/code/python/appname/media/uploads/images/bib_fort\xfcna.png', 54, 55, 'ordinal not in range(128)')

In this case, the Python `os` machinery expects everything on the filesystem to be an ascii string, instead it’s getting a unicode representation of a _[lower case u with umlaut](http://www.fileformat.info/info/unicode/char/00fc/index.htm "FileFormat.info")_. I did my usual checks, database character set and collation are utf8 and utf8\_unicode\_ci respectively, template character sets are utf8 too. There were apparently no problems at the application level.

In our case, it turns out Apache was running under the wrong environment. The important environment variable here is `LC_ALL`—Apache understood the value to be `POSIX` but it needs to be `en_GB.UTF-8` to allow special characters in filenames. We tried everything; **setEnv** in Apache config; exporting the variable in **bash**.

## The Solution

The only thing that would work was a direct edit to the Apache init script. On our SUSE Linux (SLES) box, we edited `/etc/init.d/apache2` and inserted the following lines before the case switches.

    LANG='en_GB.UTF-8';
    LC_ALL='en_GB.UTF-8';
    LC_LANG='en_GB.UTF-8';
    export LANG;
    export LC_ALL;
    export LC_LANG;

Later in the script I temporarily added `echo $LC_ALL;` to verify the environment variable had indeed been set. Restarting Apache via the init script resulted in successful uploads of files named using accented or extended character sets.

As ever, we are indebted to [Stack Overflow](http://stackoverflow.com/questions/4398540/unicodeencodeerror-when-saving-imagefield-containing-non-ascii-characters-in-djan "Apache - UnicodeEncodeError when saving ImageField containing non-ASCII characters in Django admin") for helping us find this solution.

