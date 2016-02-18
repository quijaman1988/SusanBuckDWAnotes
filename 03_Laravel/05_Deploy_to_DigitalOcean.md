# Deploy
With your Laravel application up and running locally, let's look at the procedure for getting it running on your live server.

These notes will cover the procedure for deploying to DigitalOcean. You should already have a Droplet running from P1, and that's the same Droplet you'll deploy to, so no need to create a new one.




## Summary
These following steps are done once, to configure your server and make it &ldquo;Laravel ready&rdquo;:

1. Install Composer on your Droplet
2. Enable `mod_rewrite`


The following steps are done any time you're deploying a new Laravel project to your server:

1. Clone your Laravel app
2. Build `vendor/` directory
3. Set permissions
4. Set up `.env` file
5. Configure subdomain




## Install Composer on your Droplet
While SSH'd into your Droplet, see if Composer is installed by running the composer command:

```bash
$ composer
```

As of this writing, the Droplet setup we're using does *not* come with Composer pre-installed so you'll likely see a message saying `No command 'composer' found`.

Installing Composer on your Droplet is straightforward...

Move into your bin directory:

```bash
$ cd /usr/local/bin
```

Use cURL to download Composer:

```bash
$ curl -sS https://getcomposer.org/installer | sudo php
```

Rename the composer executable to just `composer` so it's simple to call:

```bash
$ sudo mv composer.phar composer
```

Test it's working:

```bash
$ composer
```
See a list of Composer commands? Good, you're ready to move to the next step...




## Enable mod_rewrite
Laravel requires Apache's `mod_rewrite` for Routing purposes.

To enable this module, run the following command on your DigitalOcean droplet:

```bash
$ sudo a2enmod rewrite
```
Restart Apache to make these this change take effect:

```bash
$ sudo service apache2 restart
```




## Clone your Laravel app
While still SSH'd into your Droplet, navigate into your web accessible directory at `/var/www/html`:

```bash
$ cd /var/www/html
```

Clone the Laravel project which you created in the last doc.

Example:
```bash
$ git clone git@github.com:susanBuck/foobooks.git
```

Navigate into the resulting `foobooks` directory:

```bash
$ cd foobooks
```




## Build vendor/ directory
If you compare the contents of your local application files to your production application files on your Droplet, you'll notice the Droplet version is missing a `vendor/` directory.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-foobooks-on-droplet-no-vendor-directory@2x.png' style='max-width:1026px; width:100%'>

This is because vendors are managed by Composer and *not* version controlled. This is configured via `.gitignore` which lists `vendor/` as a directory to ignore:

<img src='http://making-the-internet.s3.amazonaws.com/laravel-vendor-in-gitignore@2x.png' style='max-width:482px; width:100%'>

Given this, you need to have Composer build your vendor's directory with this command:

```bash
$ composer install --prefer-dist
```

When that command is complete (it may take a few minutes), if you view the contents of your live app, you should now see a `vendor/` directory.




## Permissions
As discussed when we were setting up Laravel on your local server, Laravel/the Apache web server needs write access to the `storage` and `bootstrap/cache` directories.

This was a step we could skip locally because of how your local servers are configured, but it can't be skipped on your live servers. On your DigitalOcean server we need to make sure Apache has write access to these folders.

To do this, we first identified that on DigitalOcean servers, Apache runs under a user called `www-data`.

Given that, make the user `www-data` own the `storage` directory and everything in it (`-R`):

```bash
$ sudo chown -R www-data storage
```

And now do the same two steps for the `bootstrap/cache` directory:
```bash
$ sudo chown -R www-data bootstrap/cache
```

<sup>
Ref: [SuperUser: Setting correct permissions for uploading files](http://superuser.com/a/581259/84723)
</sup>




## Set up .env file on production
Like the `vendors/` folder, the `.env` file is also listed in `.gitignore` so it's also excluded from version control. Because of this, you need to manually create a `.env` file on your live server in order for your application to work there.

To do this, use nano to create/open a file on your live server at `/var/www/html/foobooks/.env`

```bash
$ cd /var/www/html/foobooks
$ nano .env
```

Inside this new blank file, paste the following code:

```
APP_ENV=production
APP_DEBUG=false
APP_KEY=longrandomstring

DB_HOST=localhost
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret

CACHE_DRIVER=file
SESSION_DRIVER=file
QUEUE_DRIVER=sync

MAIL_DRIVER=smtp
MAIL_HOST=mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
```

After you paste this code, save and exit nano (`ctrl` + `x`, then `y`, then `Enter`).

The above code is just a copy of the default local .env file except `APP_ENV` was changed from `local` to `production`, and `APP_DEBUG` was changed from `true` to `false`.

One final step&mdash; you may have noticed that `APP_KEY` above was set to `longrandomstring`. To create a better app key, which is used for encryption purposes throughout your app, run this command:

```bash
$ php artisan key:generate
```

(If you're curious, you can `cat .env` to see the new key that was generated)

Note: Later we'll discuss environments in full details and explain what exactly .env is doing and how it works.


## Configure subdomain
To access your Laravel application from the web, you'll want to set up a subdomain that points to it. For this you will follow the same procedure you did to create `http://helloworld.yourdomain.com` and `http://p1.yourdomain.com`.

For clarity's sake, let's outline the procedure again...

Find your DNS settings from your domain provider and create a new domain that points to your Droplet's IP Address. In our example, we're setting up `http://foobooks.dwa15-practice.biz`:

<img src='http://making-the-internet.s3.amazonaws.com/laravel-foobooks-subdomain@2x.png' class='' style='max-width:1260px; width:100%' alt=''>

Next, you need to set up a VirtualHost record for this new domain. If you'll recall, this is done in the `/etc/apache2/sites-enabled/` directory on your Droplet in a file called `000-default.conf`.

At the end of this file, *after* any existing VirtualHost blocks you may already have, add a new one:

```html
<VirtualHost *:80>
	ServerName foobooks.dwa15-practice.biz
	DocumentRoot "/var/www/html/foobooks/public"
	<Directory "/var/www/html/foobooks/public">
		AllowOverride All
	 </Directory>
</VirtualHost>
```

**IMPORTANT:** For Laravel apps, the root needs to point the the `public` folder within your application. Note how that's done above for `ServerName` and `DocumentRoot`.

Here's an example of what our `000-default.conf` file looked like after adding this:

<img src='http://making-the-internet.s3.amazonaws.com/laravel-foobooks-virtualhost@2x.png' class='' style='max-width:853px; width:100%' alt=''>

After you save your changes to `000-default.conf`, restart Apache to make the change take effect:

```bash
$ sudo service apache2 restart
```




## Test it out
Your Laravel app should now be running on your subdomain.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-foobooks-on-droplet@2x.png' style='max-width:592px; width:100%'>




## Moving forward:
That was a lot of steps we took to get your server ready for Laravel. Keep in mind, though, that most of the above steps only need to happen a) on a new server or b) after a new Laravel app install.

Once setup is complete, your process for deploying changes can be summarized in these three steps:

1. From local `add`, `commit`, and `push` changes.
2. SSH into your DigitalOcean droplet and navigate into your app folder, then run `git pull origin master`.
2. Also while SSH'd in to your app folder, run `composer install` to install any dependencies.




## Common problems
When running your application on your production server, the most common symptom you'll see when there's a problem is a blank white screen, or a generic error message that says `Whoops, there was a problem`.

Neither of these outcomes gives you much useful information to work with, so the first place you'll want to check for clues is in the Laravel log file located in `storage/logs/laravel.log`.

Common issues we see with first time installations include the following:

1. Lack of a `.env` file.
2. Forgetting to build the `vendors` directory.
3. Forgetting to set the necessary write permissions on `storage` and `bootstrap/cache`

If your application won't run, double check you've completed the above 3 steps correctly.
