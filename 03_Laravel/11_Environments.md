See also: <https://laravel.com/docs/configuration#environment-configuration>

---

## Purpose of environments
Building a modern web application often entails running that application in different environments that have different configuration needs.

For example, when running your application locally on your machine, you would define this as a `local` environment and it would have specific configuration needs:

+ Turn on all error reporting
+ Connect to a local, development database
+ Route outgoing mail through a service like [MailTrap.io](https://mailtrap.io/)

On the flip side, you also have your application running on a live server, which you would define as a `production` environment that may have these specific configuration needs:

+ Turn off all error reporting to the page
+ Connect to a *live* database
+ Send outgoing mail using a service like [SendGrid](https://sendgrid.com/)

`local` and `production` are the two most common environments, but you may have other ones as well:

+ Staging
+ Testing
+ Different developers may customize how their own local environments run


## How they work
Laravel utilizes the [DotEnv](https://github.com/vlucas/phpdotenv) PHP library by Vance Lucas.

Each environment in which you run your application has a `.env` file that sets configurations specific to that environment. The `.env` file is not stored in version control, as the contents will vary across different environments.

__Shared configurations__, that is configurations that are common to all environments, are defined in `config/`. Each file in your applications `config/` directory, and it is organized into categories (`app`, `auth`, `database`, `mail`, etc). Each configuration file returns an array of configs.

For example, if you open `/config/mail.php` you'll see the following contents (comments have been removed):

```php
<?php

return [

    'driver' => env('MAIL_DRIVER', 'smtp'),
    'host' => env('MAIL_HOST', 'smtp.mailgun.org'),
    'port' => env('MAIL_PORT', 587),
    'from' => ['address' => null, 'name' => null],
    'encryption' => env('MAIL_ENCRYPTION', 'tls'),
    'username' => env('MAIL_USERNAME'),
    'password' => env('MAIL_PASSWORD'),
    'sendmail' => '/usr/sbin/sendmail -bs',

];
```

Note how this file returns an array of configurations, where the __key__ is the __configuration name__, and the __value__ is the __configuration setting__.


In addition to examining your configuration files, you can see what specific configs are set to using the the global `config` helper function.

For example:

```php
Route::get('/practice', function() {
    echo config('mail.driver');
    echo config('app.url');
});
```




## Overwriting defaults
If you want to override configs on an environment by environment level, that is, from within the `.env` file, you have to explicitly state this in your config files.

For example, in `/config/app.php`, `debug` is set to read `APP_DEBUG` if it exists in the .env file; if it does not, it defaults to false.

```
'debug' => env('APP_DEBUG', false),
```

Contrast this to `url`, which is *not* taking the `.env` setting into account:

```
'url' => 'http://localhost',
```

This means that even if you set `APP_URL` in your `.env` file, it would have no effect. You would have to update `url` in `app.php` to look like this:

```
'url' => env('APP_URL','http://localhost'),
```

This says the `url` will be whatever `APP_URL` is set to in the `.env` file, and if `APP_URL` is not set, it will default to `http://localhost`.




## What's my current environment?
You can find out the current environment that your application is running in with this command:

```
echo \App::environment();
```

You can also have Artisan tell you:

```
$ php artisan env
Current application environment: local
```

How does Laravel know which environment you're running in? Check out `bootstrap/environment.php`:

```php
$env = $app->detectEnvironment(function()
{
    return getenv('APP_ENV') ?: 'production';
});
```

Note how the default environment is `production` if one is not set.




## Use of `.env.example`
Laravel comes with a `.env.example` file. While this file is not actually used anywhere, it can serve as a good template for suggested settings for your app.

This way, if you deploy to a new server, or a new developer clones your app to begin working on it, they can use `.env.example` as a starting point for their own `.env` file.
