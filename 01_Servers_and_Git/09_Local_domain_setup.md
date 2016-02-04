Within your local system, you'll eventually want to work on many different applications, sometimes even at the same time. Given this, it can be a drag to have to reconfigure your DocumentRoot for http://localhost every time you want to switch between apps.

In the following notes we'll set up __Virtual Hosts__ for individual apps so that you can have local URLs specific to each app you're working on.

With these instructions you could set it up so that...

+ `http://helloworld.yourdomain.com` has a local equivalent at `http://helloworld.loc`
+ `http://foobooks.yourdomain.com` has a local equivalent at `http://foobooks.loc`
+ `http://p1.yourdomain.com` has a local equivalent at `http://p1.loc`
+ `http://p2.yourdomain.com` has a local equivalent at `http://p2.loc`
+ `http://p3.yourdomain.com` has a local equivalent at `http://p3.loc`
+ `http://p4.yourdomain.com` has a local equivalent at `http://p4.loc`

In the notes below, we'll set up `http://foobooks.loc` as our example.




## 1. Tell Apache to use the virtual hosts file

First, locate and open `httpd.conf`, your local server's Apache configuration file.

* Mac: `/Applications/MAMP/conf/apache/httpd.conf`
* Windows: `c:\xampp\apache\conf\httpd.conf`

In the `httpd.conf` file, make sure the line including `httpd-vhosts.conf` is *not* commented out by removing the pound sign from the start of the `Include` (if there is one):

Mac:

```txt
# Virtual hosts
Include /Applications/MAMP/conf/apache/extra/httpd-vhosts.conf
```

Windows:

```txt
# Virtual hosts
Include conf/extra/httpd-vhosts.conf
```


## 2. Create a new host

Next, open your computer's hosts file. This file can be used to route domains to an IP address of your choice.

* Mac: `/private/etc/hosts`
* Windows: `c:/Windows/System32/drivers/etc/hosts`

(Note, there's no extension on this file.)

The hosts file is protected, so you'll need to open it with administrator privileges.

Mac:

```bash
$ sudo nano /private/etc/hosts
```

Windows/Cmder:

```bash
$ elevate nano c:/Windows/System32/drivers/etc/hosts
```

At the bottom of your hosts file, add a new host:

```txt
127.0.0.1 foobooks.loc
```

This is telling your computer that whenever you access `http://foobooks.loc` from your computer, it should map to the ip address `127.0.0.1` (the IP address of your local server).


## 3. Virtual Host entry

Next, tell your local server how to handle requests to `http://foobooks.loc` via a VirtualHost entry.

Open your `httpd-vhosts.conf` file.

Mac:

```bash
$ sudo nano /Applications/MAMP/conf/apache/extra/httpd-vhosts.conf
```

Windows/Cmder:

```bash
$ elevate nano c:\xampp\apache\conf\extra\httpd-vhosts.conf
```

At the bottom of this file you'll see two example virtual host blocks, one for `dummy-host.example.com` and another for `dummy-host2.example.com`. __Delete these.__

<img src='http://making-the-internet.s3.amazonaws.com/vc-vhost-examples@2x.png' class='' style='max-width:728px; width:75%' alt=''>

Now, add your own VirtualHost block:

Mac/MAMP users, yours might look something like this:
```txt
<VirtualHost *:80>
    ServerName foobooks.loc
    DocumentRoot /Applications/MAMP/htdocs/foobooks/public
    <Directory /Applications/MAMP/htdocs/foobooks/public>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Order allow,deny
        allow from all
    </Directory>
</VirtualHost>
```

Windows/XAMPP users, yours might look something like this:
```txt
<VirtualHost *:80>
    ServerName foobooks.loc
    DocumentRoot c:\xampp\htdocs\foobooks\public
    <Directory c:\xampp\htdocs\foobooks\public>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Order allow,deny
        allow from all
    </Directory>
</VirtualHost>
```

Adjust the following as needed:

1. `ServerName` (use `.loc` or `.dev` to distinguish it from the live TLD)
2. `DocumentRoot`
3. `Directory` (same as `DocumentRoot`)

Note, the above VirtualHost blocks assumes you're running on Port 80 (`*:80`). If you're running your local Apache on a different port, make that edit.

This is what your `httpd-vhosts.conf` file might look like when you're done:

<img src='http://making-the-internet.s3.amazonaws.com/vc-vhosts-done@2x.png' class='' style='max-width:726px; width:75%' alt=''>

**Restart your local server** and test out your local URL.

Make sure you explicitly type in `http://foobooks.loc` with `http://` at the beginning. If you don't, your browser may just try and do a web search for `foobooks.loc` because it does not recognize `.loc` as a domain extension.




## Summary

To be repeated every time you want to add a new app):

+ Add a new local URL in your computer's `host` file.
+ Add a new `<VirtualHost>` record block in MAMP/Apache's `httpd-vhosts.conf` file.

Note how it's only Steps 2 and 3 above that need to be repeated for any new apps. Step 1 is a one time deal to get virtual hosts working.




## Tips
* Make shortcuts to `httpd-vhosts.conf` and `hosts` for quick access in the future.
* [MAMP Pro ($59)](http://www.mamp.info/en/mamp-pro/) offers a point and click interface to quickly edit hosts, in addition to [other features](http://www.mamp.info/en/mamp-pro/features/matrix.html).
* If you run into any problems, check your Apache error log.
