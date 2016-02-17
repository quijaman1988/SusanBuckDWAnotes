## What is the PATH variable?
When you run a command for a program in command line, it looks for the corresponding executable files using the directories listed in your *PATH variable* as a map. The *PATH variable* is one of many *Environment Variables* your computer uses.

Given this, when you add a new executable/program you wish to use via command line, you have to add its directory to your PATH variable.


## Mac: Edit your PATH
To start, use this command to see what your PATH variable is currently set to:

```bash
echo $PATH
```

One way to add new paths to your PATH variable is to add them in the `/etc/paths` file.  

Let's use an example scenario of adding the PHP executable that comes with MAMP to your PATH. This will make it so that when you run the command `php` from command line, it will use MAMP's PHP executable.

This semester, the version of PHP we're using with MAMP is `php5.6.10` ([screenshot](http://making-the-internet.s3.amazonaws.com/sysadmin-identify-latest-version-of-php-in-mamp.png)).

If you dig into the MAMP folder contents on your computer, you can find a PHP executable in `/Applications/MAMP/bin/php/php5.6.10/bin`, so that's the path we want to add.

<img src='http://making-the-internet.s3.amazonaws.com/sysadmin-finding-php-in-mamp@2x.png' style='max-width:1121px; width:100%' alt='Finding PHP in MAMP'>

Using Nano and the `sudo` command, open `/etc/paths`:

```bash
$ sudo nano /etc/paths
```

At the *start* of `/etc/paths`, add this line:

```bash
/Applications/MAMP/bin/php/php5.6.10/bin
```

<img src='http://making-the-internet.s3.amazonaws.com/sysadmin-append-to-paths@2x.png' style='max-width:1136px; width:100%' alt=''>

Save your changes (`ctrl` + `x`, then `y`, then *Enter*).

Close and restart Terminal. (This step is important! If you don't do this, your changes won't take effect)

Test it out:

```bash
# Where is php?
$ which php

# What version?
$ php -v
```

Expected output:
```bash
$ which php
/Applications/MAMP/bin/php/php5.6.10/bin/php
$ php -v
PHP 5.6.10 (cli) (built: Jul  6 2015 14:28:54)
Copyright (c) 1997-2015 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2015 Zend Technologies
```


__Tips__

Instead of editing `/etc/paths` you can also prepend your MAMP PHP path to the PATH variable by adding the following line to your `~/.bashrc` file:
```bash
export PATH="/Applications/MAMP/bin/php/php5.6.10/bin":$PATH
```





## Windows: Edit your PATH

To begin, use this command to see what your PATH variable is currently set to:

```bash
PATH
```

PATH variables in Windows are set via *My Computer* > *Properties* (alternatively, you can get here by running `sysdm.cpl` from Command Line)

Then, from the *Properties* screen, go to *Advanced* > *Environment Variables* > *Path*.

You'll find `Path` under *System variables*. Select it and click *Edit*.

<img src='http://making-the-internet.s3.amazonaws.com/sysadmin-windows-getting-to-path@2x.png' style='max-width:925px; width:100%' alt=''>

Let's use an example scenario of adding the PHP executable that comes with XAMPP to your PATH. This will make it so that when you run the command `php` from command line, it will use XAMPP's PHP executable.

XAMPP includes a php executable located at `C:\xampp\php\`:

<img src='http://making-the-internet.s3.amazonaws.com/sysadmin-finding-php-in-xampp@2x.png' style='max-width:772px; width:100%' alt='Finding PHP in XAMPP'>

...so that's the path we want to add.

From within the Environment Variables screen we navigated to above, edit your `Path` *System Variable* by appending `c:\xampp\php\` to the end.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-setting-path-variable-on-windows@2x.png' class='' style='max-width:972px; width:100%' alt='Editing your PATH in Windows'>

Ok/Save your changes.

Restart Cmder.

Test it out:

```bash
# Where is php?
$ where.exe php

# What version?
$ php -v
```

Expected output:
```bash
C:\Users\Susan
$ where.exe php
C:\xampp\php\php.exe
C:\xampp\php\php.exe

C:\Users\Susan
$ php -v
PHP 5.6.15 (cli) (built: Oct 29 2015 12:40:36)
Copyright (c) 1997-2015 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2015 Zend Technologies
```


## Reference
+ [Mac OSX Change Path Variables](http://architectryan.com/2012/10/02/add-to-the-path-on-mac-os-x-mountain-lion/#.U9nTwIBs9vI).
+ [SO: Adding directory to PATH Environment Variable in Windows](http://stackoverflow.com/questions/9546324/adding-directory-to-path-environment-variable-in-windows)
+ [How to set path and environment variables in Windows](http://www.computerhope.com/issues/ch000549.htm)
