---
layout: post
title: Connecting to Microsoft SQL Server from UNIX (Linux/Mac OSX) in Python
date: '2013-02-15T11:53:00+00:00'
tags: []
tumblr_url: https://richbs.org/post/43142767072/connecting-to-microsoft-sql-server-from-unix
---
Assuming you have a username and password with some kind of access to a MSSQL server, this might help you.

Mac OS X instructions lower down but the full post should help it all sink in.

# Linux

## Install unixODBC, this abstracts database access

If I was on SUSE Linux (SLES) I would hit up `yast` and install the following packages

    $ su root
    $ yast
    
    unixODBC │2.2.11 │2.2.11 │ODBC driver manager with some drivers included │
    unixODBC-devel │2.2.11 │2.2.11 │Includes and Static Libraries for ODBC Development

## Install FreeTDS driver

This is the Open Source MSSQL Driver for Unix-based systems: [http://www.freetds.org/](http://www.freetds.org/)

    wget ftp://ftp.freetds.org/pub/freetds/stable/freetds-stable.tgz
    tar xzvf freetds-stable.tgz 
    cd freetds-0.XX/

Next, configure FreeTDS with the location of unixODBC, you’re telling it you plan to use it via an ODBC interface, if /usr/lib/unixODBC exists then the value you need is /usr. Then, you can `make` and `install`.

    ./configure --with-unixodbc=/usr --with-tdsver=8.0
    make
    sudo make install

Find your freetds.conf, we’re going to stick in the connection details for your SQL Server Box. On this Linux box, it’s /usr/local/etc/freetds.conf

In the stock conf, there’s a helpful example provided:

    # A typical Microsoft server
    [egServer70]
        host = ntmachine.domain.com 
        port = 1433
        tds version = 7.0

I added this line to this server config…

    client charset = UTF-8

I can’t tell you how awesome this line is. It uses the iConv libraries to make sure you’re getting UTF-8 data over your ODBC connection, even if the SQL Server config is some godawful WIndows cp1252 ANSI code page.

Create your own server config with your own credentitals. The tds version relates to the version of SQL server you are using [http://www.freetds.org/userguide/choosingtdsprotocol.htm.](http://www.freetds.org/userguide/choosingtdsprotocol.htm.) In short, if you’re using SQL Server 2000 and above, use version 8.0

Port 1433 is the default port for connecting to MSSQL Servers, why not check if you can connect from your given server before trying the FreeTDS driver

    $ telnet IP_OR_MSSQL_SERVER_NAME 1433

FreeTDS installs the `tsql` binary for testing your connections. The `-S` parameter refers to the `[egServer70]` section of `freetds.conf`:

    $ tsql -S egServer70 -U niceuser
    locale is "en_GB.UTF-8"
    locale charset is "UTF-8"
    Password: 
    1>

Great! That `1>` prompt smells of success!

## unixODBC

[http://www.unixodbc.org](http://www.unixodbc.org)

Now we need to register the driver with unixODBC, using the odbcinst binary that is installed with unixODBC

Create a file:

    touch freetds-driver

edit the file - the key detail here is the Driver line. It should point to wherever the FreeTDS installation wrote the libtdsodbc.so file. In most cases, following these instructions on a Linux box, it will be in /usr/local/lib but it could vary on different UNIX-based systems.

    [FreeTDS]
    Description = TDS driver (Sybase/MS SQL)
    Driver = /usr/local/lib/libtdsodbc.so

Register the FreeTDS driver with ODBC

    sudo odbcinst -d -i -f freetds-driver

So that’s the driver registered, now we need to let ODBC know about the SQL Server we want to connect to, as set out in `freetds.conf`

Add a DSN for the SQL Server in odbc.ini

    sudo vim /etc/unixODBC/odbc.ini
    
    [nicedcn]
    Driver = FreeTDS 
    Description = MSSQL database for my nice app
    # Servername corresponds to the section in freetds.conf
    Servername=egServer70
    Database = NICE_APP

`isql` is the client installed by unixODBC. Pass `isql` the DSN you defined along with your username and password

    $ isql -v nicedsn niceuser nicepass 
    
    
    +---------------------------------------+
    | Connected! |
    | |
    | sql-statement |
    | help [tablename] |
    | quit |
    | |
    +---------------------------------------+
    SQL>

Now ODBC is allowing you to enter plain old SQL, **INTO A MSSQL SERVER, FROM A LINUX BOX!**

    SQL> SELECT * FROM tablename

If you encounter issues, make sure that isql knows where to find your odbc ini file or it will not be able to. I’ve found environment variables to be useful in this process, e.g.:

    ODBCINI=/etc/unixODBC/odbc.ini ; export ODBCINI

## Install pyodbc

[http://code.google.com/p/pyodbc/](http://code.google.com/p/pyodbc/)

To compile it you will need the GNU C++ compiler. Under SLES YAST, it is called `gcc-c++` but elsewhere you might find it as `g++`.

You’ll also need the `python-devel` and `unixODBC-devel` package to compile against. Check for these in your distribution’s package manager.

    wget http://pyodbc.googlecode.com/files/pyodbc-x.x.x.zip
    unzip pyodbc-x.x.x.zip 
    cd pyodbc-x.x.x/
    python setup.py build
    sudo python setup.py install

Or, in modern parlance:

    sudo pip install pyodbc

# MAC OSX

There are slight differences on this platform. You’ll need XCode from the Mac App Store and you’ll need to install the **Command Line Tools** from `Xcode`\>`Preferences`\>`Downloads`\>`Components`.

    tar xzvf freetds-stable.tgz 
    cd freetds-0.XX/
    ./configure
    make
    sudo make install

Then, edit freetds.conf lie

    $ vim /usr/local/etc/freetds.conf

Pop in your SQL server details

    [PEODB]
    host = 172.16.XX.XXX
    port = 1433
    tds version = 8.0
    # this is a great setting to make sure that data reaches you in UTF-8
    # chances are it's in some godawful Windows ANSI Code Page
    client charset = UTF-8

iODBC takes responsibility for the ODBC legwork on Mac OS X Python, you can edit the config and create the DSN you need. The `Driver` line is essential to tell iODBC where the freetds driver is.

    $ vim /etc/odbc.ini

Here are the contents:

    [nicedsn]
    Description = Live Server for My Nice App
    Driver = /usr/local/lib/libtdsodbc.so
    Servername = 172.16.XX.XXX
    Database = NICE_APP
    UserName = niceuser
    Password = nicepass

You’ll need to have `pyodbc` for Python installed:

    wget http://pyodbc.googlecode.com/files/pyodbc-x.x.x.zip
    unzip pyodbc-x.x.x.zip 
    cd pyodbc-x.x.x/
    python setup.py build
    sudo python setup.py install

Or, in modern parlance:

    sudo pip install pyodbc

Then get on the Python interpreter:

    $ python
    …
    >>> import pyodbc
    >>> conxn = pyodbc.connect('DSN=nicedsn;UID=niceuser;PWD=nicepass;')
    >>> conxn
    <pyodbc.Connection object at 0x93660>
    >>> cur = conxn.cursor()
    >>> cur.execute("SELECT TOP 10 * FROM Events")
    <pyodbc.Cursor object at 0x104d9a690>
    >>> cur.fetchOne()
    (1, 1, 1858, 1858, 3, 5, datetime.datetime(2008, 6, 23, 11, 0), datetime.datetime(1900, 1, 1, 0, 0), datetime.datetime(2008, 6, 23, 9, 0), 10, 2, 0, '', 0, '', 0, datetime.datetime(2008, 6, 21, 11, 23, 13, 813000))

Look at those lovely native Python `datetime` instances!

## References

These references are invaluable:

- [http://www.easysoft.com/developer/languages/python/pyodbc.html](http://www.easysoft.com/developer/languages/python/pyodbc.html)
- [http://www.unixodbc.org/doc/FreeTDS.html](http://www.unixodbc.org/doc/FreeTDS.html)
