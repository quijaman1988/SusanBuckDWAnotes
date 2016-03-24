## What is namespacing?

<http://php.net/manual/en/language.namespaces.rationale.php>:

> *What are namespaces? In the broadest definition namespaces are a way of **encapsulating items**. This can be seen as an abstract concept in many places. For example, in any operating system directories serve to group related files, and act as a namespace for the files within them. As a concrete example, the file `foo.txt` can exist in both directory `/home/greg` and in `/home/other`, but two copies of `foo.txt` cannot co-exist in the same directory. In addition, to access the `foo.txt` file outside of the `/home/greg` directory, we must prepend the directory name to the file name using the directory separator to get `/home/greg/foo.txt`. This same principle extends to namespaces in the programming world.*


Namespacing is a principle in programming that allows us to prevent **name collisions**. Name collisions occur when two different packages of code try to use the same class names.

For example, let's imagine two hypothetical packages of code your application might want to implement:

+ The first is a package called *Texter* that lets you send text notifications to your user.
+ The second is a package called *AutoCaller* that lets you send automated voice messages to your user.

Both of these packages have a class called `Message`.

So if you write...

```php
$message = new Message();
```

...how does your code know *which* Message class you're trying to use - the one from *Texter* or the one from *AutoCaller*?

The solution to this problem is __namespacing__. Using namespacing, you can specify in more detail *which* Message class you're trying to use.

Here's a basic example:

```php
$message1 = new Texter\Message;
$message2 = new AutoCaller\Message;
```


## Autoloading
Not only does namespacing help prevent name collisions, it also helps power [__autoloading__](http://php.net/manual/en/language.oop5.autoload.php).

Autoloading is a feature PHP has which makes it possible to invoke classes in your code without having to explicitly require the corresponding class files.

In other words, without autoloading you might have to write:

```
require('texter/Message.php');
$message1 = new Texter\Message;
```

...but with autoloading, you can simply write:

```
$message1 = new Texter\Message;
```

...and PHP will automatically load the corresponding `Message.php` from the Texter directory (if it hasn't already).

In order for autoloading to work, you have to first give your autoloading function some details about *where* to find your classes.

In Laravel, there's some autoloading configured in `composer.json` :

```
"autoload": {
    "classmap": [
        "database"
    ],
    "psr-4": {
        "App\\": "app/"
    }
},
```

This is using 2 different methods of setting up your autoloading:

1. By `classmap`. This is simply autoloading by directory. It's saying &ldquo;*Look in the `database/` directory, you'll find classes there*&rdquo;.

2. By `psr-4`. This is where namespacing and autoloading join forces. To explain this, let's first break down what a PSR is.




## PSR
PHP used to be a very haphazard community. Different frameworks and developers were following various different methods for writing their code.

As PHP has matured, though, it has become more organized and as a result, there was a consortium formed&mdash; that is a group of people who get together and study, discuss, debate, and vote on the &ldquo;best way to do things.&rdquo;

This consortium is called the [__FIG - Framework Interoperability Group__](http://www.php-fig.org/). As the name implies, the purpose of this group is to decide the best way to handle routine programming needs, so that there is some consistency amongst frameworks.

One of the things that FIG has done is to develop best practices called __PSR - PHP Standard Recommendations__.

Currently there are 5 active PSRs:

+ [PSR-1: Basic Coding Standard](http://www.php-fig.org/psr/psr-1/)
+ [PSR-2: Coding Style Guide](http://www.php-fig.org/psr/psr-2/)
+ [PSR-3: Logger Interface](http://www.php-fig.org/psr/psr-3/)
+ [PSR-4: Autoloading Standard](http://www.php-fig.org/psr/psr-4/)
+ [PSR-6: Caching Interface](http://www.php-fig.org/psr/psr-6/)
+ [PSR-7: HTTP Message Interface](http://www.php-fig.org/psr/psr-7/)

The PSRs specific to namespacing/autoloading is PSR-4, so that's the one we're going to focus on.



## PSR-4
What PSR-4 does is creates a correlation between your class names and the location of your class files.

So, for example if you had PSR-4 configured and used this class setup:

```php
namespace Texter;

class Message() {

}
```

...your autoloader would look for this class at `Texter\Message.php`.




## PSR-4 in Laravel
Looking at `composer.json` in Laravel 5 we see this PSR-4 setting:

```json
"psr-4": {
    "App\\": "app/"
}
```

The key `App` is the namespace and `app/` is the file path.

The `\\` after App is used to prevent name collisions with other namespaces like, for example, `Apples` or `Application`.

This key-value pair indicates that when we use a namespace that starts with `App`, the autoloader should look for the corresponding file in `app/`.

You can see this in action in our `BookController.php`.

At the top, the namespace is set to:

```
namespace App\Http\Controllers;
```

And the path to the file corresponds at:

```
app/Http/Controllers/BookController.php
```

Note that if you had changed your App name after first installing Laravel `php artisan app:name Foobooks`, that would be reflected here:

```json
"psr-4": {
    "Foobooks\\": "app/"
}
```


The `app` folder is just one of the folders that is PSR-4 autoloaded in Laravel.

There's more autoloading going on under the hood by default with Composer&mdash; for example everything in the `/vendor` directory is autoloaded.




## Summary
Namespacing is an important concept in modern PHP and Laravel. Understanding the basics outlined above will help you as you navigate the Laravel codebase, start including external packages, and write your own code.

Summary: Namespacing...

1. helps avoid name collisions.
2. helps power autoloading by mapping namespaced class names to file paths.




## Namespacing in Controllers

Early on, we saw examples of Laravel facades in the `routes.php` file like this:

```php
# Print out the current environment
echo App::environment();
```

...or this:

```php
# Generate a hashed password
Hash::make('secretpassword');
```


If you try those same invocations in a controller action...

```php
public function getIndex() {
    echo App::environment();
}
```

...it'll fail with the following error:

```txt
FatalErrorException in BookController.php line 16:
Class 'App\Http\Controllers\App' not found
```

The reason it works in the `routes.php` file is because that file is in the __global namespace__.

When you move into the controller, you're then in that controller's namespace.

The error message points this out&mdash; it's trying to find `App` in the `App\Http\Controllers\` namespace, where it does not exist.

To fix this, you can put a backward slash in front of the facade invocations, forcing it to look for `App` in the global namespace:

```php
public function getIndex() {
    echo \App::environment();
}
```

Alternatively, you could add a `use` statement at the top of your controller:

```php
use App;
```

The same rule applies when using the Debugbar alias we set up when discussing Packages:

```php
public function getIndex() {
    echo Debugbar::info('Testing...');
}
```

You could prefix all your Debugbar calls with `\`, or, add the following `use` statement at the top of your controller:

```
use Debugbar;
````

Note that the namespace is not `Barryvdh\Debugbar` because of the alias setup for `Debugbar` in `config/app.php`.
