## What is a package?

Packages are a term used in Composer to describe bundles of softwares that developers create and share.

Laravel itself is a package (that depends on many other packages).

Packages save you time and effort. Here are some example scenarios:

+ Need to resize your user's profile photos? [There's a package](https://github.com/Intervention/image) that gives you all sorts of PHP image resizing functionality including cropping, resizing, converting formats, etc.
+ Need to output PDF receipts for your users? [There's a package](https://github.com/cangelis/php-pdf) for converting HTML to PDFs.
+ Need to distinguish your users by roles such as admin, moderator, guest, etc.? [There's a package](https://github.com/romanbican/roles) for managing roles and permissions.





## Finding and choosing packages

The main source for researching/browsing packages is [Packagist](https://packagist.org/).

There's also a Laravel-specific package browsing site called [Packalyst](http://packalyst.com/packages).

This brings up some interesting points:
+ Packages are not Laravel specific.
+ Some packages are designed to work specifically with Laravel.
+ Other packages are more generic and can work with any project, including Laravel.
+ Some packages are designed for other frameworks, and won't play nice with your Laravel project.

These are factors you'll want to consider when choosing which packages you'll use.

### Compatibility
+ Will this package work with my Laravel app?
+ Specifically, will it work with the version of Laravel I'm using?
+ If it was not designed for Laravel, can I still use it?

### Quality
+ Does it have a lot of downloads?
+ Is the documentation clear and well written? The quality of the documentation/README.md file is usually a good reflection of how well written the package itself is written.
+ Has it been recently updated or is it several years old?
+ Is it being recommended by a reputable source? (Like Laracasts.com).

When browsing for a package, it doesn't necessarily have to pass *all* of these tests. For example, there are many good packages that are several years old and still get the job done. But if it's old, *and* the documentation is terrible, *and* there's only 10 downloads...you might want to stay away.

### Test driving
If you come across several quality options that may help you accomplish the task at hand, you may want to &ldquo;test drive&rdquo; the different packages.

These are questions you might ask yourself when comparing packages:
+ Is one package easier to use than another?
+ Is one package written in a code style you prefer more vs another?
+ Is one package lacking the features you need?
+ Is one package packed with more features than you actually need?
+ Is one package better integrated with Laravel vs another?


## Learning more about a package
When searching for packages on Packagst or Packalyst, you'll enter keywords for the kind of feature you're looking to add, for example you might type in &ldquo;image&rdquo; if you needed a package for basic image manipulation (cropping, resizing, etc.):

<img src='http://making-the-internet.s3.amazonaws.com/laravel-searching-for-image-package@2x.png' style='max-width:823px; width:100%' alt=''>

From there, you can choose matches that look good to learn more.

The main thing you want to find in the information page is the link to the Package's Github page, because that's where the most comprehensive info is.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-getting-to-github-page-from-packagist@2x.png' style='max-width:838px; width:100%' alt=''>

The Github page should include details such as:

+ General description of what the package does
+ Any requirements (for example, a certain version of PHP)
+ List of features
+ Installation instructions
+ Usage instructions
+ Examples




## Installing a package
For our examples we're going to install a few different helper packages that are going to add some convenience features to your Laravel application.

The first example is a package called __laravel-debugbar__:

+ <https://packagist.org/packages/barryvdh/laravel-debugbar>
+ <https://github.com/barryvdh/laravel-debugbar>

To install this package, we'll use the `composer require` command, followed by the full name of the package (including the vendor):

```bash
$ composer require barryvdh/laravel-debugbar
```

How did we know to use the name `barryvdh/laravel-debugbar`?

There's two ways:

First, the installation instructions on <https://github.com/barryvdh/laravel-debugbar> tell you.

Second, if you do `composer search "Laravel debug bar"` you'll see it as the third option:

```bash
$ composer search "laravel debugbar"
laravel/framework The Laravel Framework.
maximebf/debugbar Debug bar in the browser for php application
barryvdh/laravel-debugbar PHP Debugbar integration for Laravel
[...etc...]
rap2hpoutre/laravel-log-viewer A Laravel log reader
barryvdh/laravel-debugbar PHP Debugbar integration for Laravel
```

Note how there's lots of different package options when you search for &ldquo;laravel debugbar&rdquo; so make sure you choose the one that matches the vendor and name you found when researching your package.

After you run the `composer require` command, if you open `composer.json`, you'll see your `require` section now includes a line for the `barryvdh/laravel-debugbar`:

```bash
"require": {
    "php": ">=5.5.9",
    "laravel/framework": "5.2.*",
    "barryvdh/laravel-debugbar": "^2.2"
},
```

Also, you should now see a `barryvdh/laravel-debugbar/` directory in your `vendors/` directory:

<img src='http://making-the-internet.s3.amazonaws.com/laravel-debugbar-in-vendors@2x.png' style='max-width:278px; width:100%' alt=''>




## Using a package
How you use a package is going to depend on how that package is built.

+ Some packages may be built specifically for Laravel and utilize Service Providers.
+ Some packages implement PSR-4 autoloading, which will make using that package easy.
+ Some packages won't implement PSR-4 autoloading, and will require a little more effort on your part.

We're going to cover a few different examples...


## Using a package built for Laravel
Continuing with the debugbar package we've already installed, we'll start by referring to the [documentation](https://github.com/barryvdh/laravel-debugbar) for how to use this package in Laravel 5:

<img src='http://making-the-internet.s3.amazonaws.com/laravel-debugbar-usage-instructions@2x.png' style='max-width:800px; width:100%' alt=''>

This instruction is a bit sparse and assumes some pre-existing Laravel knowledge, so let's expound:

The first but in the instructions is telling you you need to add...

```php
Barryvdh\Debugbar\ServiceProvider::class,
```

...to the end of the `providers` array in the `config/app.php` file.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-add-new-service-provider@2x.png' style='max-width:839px; width:100%' alt=''>

In Laravel, a [__Service Provider__](http://laravel.com/docs/master/providers) is an organizational tool that lets you group together related classes. Service Providers are loaded when your app &ldquo;boots up&rdquo;, making those services available for your use. Service Providers are a key part of how Laravel works, and we're just scratching the surface here in terms of definition, but it's enough to continue with this example...

The second bit in the instructions is telling you to add the line...

```php
'Debugbar' => Barryvdh\Debugbar\Facade::class,
```

...at the end of the `aliases` array, also in `config/app.php`.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-add-new-alias@2x.png' style='max-width:474px; width:100%' alt=''>

That should be it to get rolling. Load any page in your app to test it out.

The Debugbar [documentation](https://github.com/barryvdh/laravel-debugbar) states:

> &ldquo;*The profiler is enabled by default, if you have app.debug=true*&rdquo;

Given this, and because you should be working locally now with debug = true, the debug bar should show at the bottom of each page by default:

<img src='http://making-the-internet.s3.amazonaws.com/laravel-debug-bar-open@2x.png' style='max-width:1000pxpx; width:100%' alt=''>

But, there's more. Further down in the Debugbar documentation, there's some *Usage* examples where it demonstrates outputting different kinds of message to the debugbar.

Let's take those instructions and test them out in a `/practice` route:

```
Route::get('/practice', function() {

    $data = Array('foo' => 'bar');
    Debugbar::info($data);
    Debugbar::error('Error!');
    Debugbar::warning('Watch out…');
    Debugbar::addMessage('Another message', 'mylabel');

    return 'Practice';

});
```

When you run the `/practice` route, if you look at the **Messages** tab of the debug bar, you should see the various outputs:

<img src='http://making-the-internet.s3.amazonaws.com/laravel-messages-in-debugbar@2x.png' style='max-width:1000px; width:100%' alt=''>

Being able to output messages like this will be extremely useful when building and debugging your applications.




## Using a generic package, not specific to Laravel

For our next example, we're going to use a Package called [__rych-random__](https://github.com/rchouinard/rych-random) which lets you easily generate random data.

Some example usages for this package might be generating random coupon codes for a e-commerce site, or random passwords if you have to reset a user's password, etc.

This particular package is not built specifically for Laravel, but it will work.


### Installing rych-random
To install rych-random, instead of using `composer require` we'll instead update `composer.json` adding this line to the `require` section:

```json
"rych/random": "dev-master"
```

After adding this line, run `composer update` to have Composer download that package.

Note the version we choose here is `dev-master` which means &ldquo;*Give us the latest version*&rdquo; rather than any specific version number.

We choose to do this because currently rych-random [only has one version tag, v0.1.0](https://github.com/rchouinard/rych-random/releases) and that version doesn't actually reflect the current build of the code. Given this, we're circumventing the versions and &ldquo;living on the edge&rdquo; so to speak.

A word of caution when doing this&mdash; if the developer of this app updates their software we'll get those updates next time we run `composer update`. Because of this, in real-world production software it is risky to use `dev-master` instead of locking into a specific version.



### Using rych-random
Taking from the notes in the [rych-random documentation](https://github.com/rchouinard/rych-random), we then set up this example:

```php
Route::get('/practice', function() {

    $random = new Rych\Random\Random();
    return $random->getRandomString(8);

});
```

When you visit `/practice` in the browser, you should see some output of a random string of letters and numbers, e.g. `ElAivf96`.

Note that in this example, we weren't guided to add any providers or aliases in `config/app.php` like we did with debugbar.

This works because in the example we're given, we're given the namespaced name for the class `Rych\Random\Random()`. Using this full namespace, Composer and our autoloading configuration knows to navigate to `vendor/rsych/` to find the appropriate class file.

But what if we didn't want to have to specify the full namespace every time?

There's two alternative options:

__Alternative Option 1__:

You can implement the `use` statement at the top of your PHP file to indicate you'll be using that class. Then, when you use it, you can simply say `new Random()`.

```php
use \Rych\Random\Random;

Route::get('/practice', function() {

    $random = new Random();
    return $random->getRandomString(8);

});
```


__Alternative Option 2__:

If you anticipate using a class frequently throughout your application, you can create a global __alias__ for it.

This is done in `/app/config/app.php` in the array called `aliases`.

Here's an example of how you'd update the `aliases` array in `/config/app.php` to add Random:

```php
'aliases' => [

    'App'       => Illuminate\Support\Facades\App::class,
    'Artisan'   => Illuminate\Support\Facades\Artisan::class,
    [...etc...]

    'Debugbar'  => Barryvdh\Debugbar\Facade::class,

    'Random'	=> 'Rych\Random\Random',

],
```

The *key* is the alias name you want to use (`Random`) and the *value* is the full path to the class, including the namespace (`Rych\Random\Random`)

Once you've set up this alias, you can call the `Random` class with no namespace and without having to utilize the `use` keyword.

```php
Route::get('/practice', function() {

    $random = new Random();
    return $random->getRandomString(8);

});
```

### Which option is best?
Which method you want to use is up to you, but here's a general guideline:

+ Using a Class just once?
    + Go with the full namespace, e.g. `$random = new Rych\Random\Random();`
+ Using a Class multiple times in a single file?
    + Set `use \Rych\Random\Random;` at the top of the file.
+ Using a Class multiple times throughout your application?
    + Create an alias.


## Practice
Using what we've covered so far, complete the following exercise:

+ Read about the package Laravel Log Viewer: <https://github.com/rap2hpoutre/laravel-log-viewer>
+ Following the given instructions, install and implement this package.




## Tips/Misc

### Issue: minimum-stability missing

Sometimes when you attempt to install a package with `composer require` you'll see a message telling you the package lacks a `minimum-stability`.

For example, you can see this with a package called [`paste\pre`](https://github.com/paste/Pre.php):

```bash
$ composer require paste/pre
[InvalidArgumentException]                                                                                           
Could not find package paste/pre at any version for your minimum-stability (stable). Check the package spelling or your minimum-stability    
```

This happens when a package is not using version tagging, and as a result Composer does not know what is the latest, stable version.

A workaround to this is to add the package by manually editing `composer.json`, setting the version of the package to `dev-master`:

```php
"require": {
    # [...existing packages...]
    "paste/pre": "dev-master"
}
```

__Once you save these changes to `composer.json` you have to run `composer update` to have it go and grab that package.__

You should avoid using packages that don't implement version numbers in important, production code. Without version numbers, you have less control over what version of the package you're getting, and it's always possible new versions could break your app without you realizing it at first.

In this case, since `pre` is a debugging tool, we could instead include it under `require-dev` and then it wouldn't matter as much that we weren't able to lock into specific versions.




## Composer update vs. Composer install

* Use `composer update` on your development environment(s) so it grabs the package versions set in your `composer.json` file and updates your `composer.lock` file to match these versions.

* Use `composer install` on your production environment(s) so it grabs the package versions set in your `composer.lock` file (i.e. mirror the versions exactly as they are on the development environment).
