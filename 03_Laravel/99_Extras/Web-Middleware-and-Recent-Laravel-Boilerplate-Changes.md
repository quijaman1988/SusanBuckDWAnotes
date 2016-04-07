## Web Middleware and recent Laravel boilerplate changes

Earlier this semester, when we first started working on the `routes.php` file in the `foobooks` example application, we saw this starting template:

```php
<?php

 /*
 |--------------------------------------------------------------------------
 | Routes File
 |--------------------------------------------------------------------------
 |
 | Here is where you will register all of the routes in an application.
 | It's a breeze. Simply tell Laravel the URIs it should respond to
 | and give it the controller to call when that URI is requested.
 |
 */

 Route::get('/', function () {
     return view('welcome');
 });

 /*
 |--------------------------------------------------------------------------
 | Application Routes
 |--------------------------------------------------------------------------
 |
 | This route group applies the "web" middleware group to every route
 | it contains. The "web" middleware group is defined in your HTTP
 | kernel and includes session state, CSRF protection, and more.
 |
 */

 Route::group(['middleware' => ['web']], function () {
     //
 });
```

When I introduced this `routes.php` file in lecture, I explained how the majority of your routes should go in that provided **web middleware group**, because that group would provide some important functionality you'd want for your web routes.

If you've installed a new Laravel application since March 23rd, you may notice two things:

1. The web middleware group is no longer included in `routes.php`
2. If you try to add it (following the example you see in foobooks) it will break things&mdash; for example, your error messages from validation will stop showing up.

The reason this is happening is because on [March 23rd](https://github.com/laravel/laravel/commit/5c30c98db96459b4cc878d085490e4677b0b67ed), Laravel made a change to the boilerplate application code such that the web middleware group is automatically applied to all your routes via your RoutesServiceProvider; as a result, the route middleware group in the routes file is no longer needed.

You'll only notice this if you do a brand new Laravel installation, not if you update your current installation (ala `composer update`). The reason is because the change was not made to the Laravel core code, but to the boilerplate code that is not impacted by updates.

So now we have a bit of a disconnect in how the web middle ware is being applied:

+ &ldquo;old way&rdquo; = Apply via group in `routes.php`
+ &ldquo;new way&rdquo; = Apply globally via `RoutesServiceProvider.php`

How this is seen across our apps so far:
+ Our class example `foobooks` is doing things via the &ldquo;old way&rdquo;.
+ If you created P3 before March 23, it's doing it the &ldquo;old way" otherwise it's doing it the &ldquo;new way&rdquo;.
+ Assuming you first create(d) P4 after March 23rd, it will do things via the &ldquo;new way&rdquo;.

To resolve this disconnect, I'm going to update the `foobooks` app to the &ldquo;new way&rdquo; and show you how to do it (below). It's totally up to you whether you want to update P3, as the &ldquo;old way&rdquo; still works perfectly fine.


## Update procedure

### Step 1)
Open `/app/Providers/RouteServiceProvider.php` and replace the existing code with [this code](https://raw.githubusercontent.com/laravel/laravel/5c30c98db96459b4cc878d085490e4677b0b67ed/app/Providers/RouteServiceProvider.php).

### Step 2)
Open `routes.php` and eliminate the web middleware group, so that all the routes currently in there are no longer encapsulated in any group.

For example, before:

```php
<?php

Route::group(['middleware' => ['web']], function () {

    Route::get('/', function () {
        return view('welcome');
    });


    Route::get('/books', 'BookController@getIndex');
    Route::get('/books/create', 'BookController@getCreate');
    Route::post('/books/create', 'BookController@postCreate');

});
```

After:
```php
<?php

Route::get('/', function () {
    return view('welcome');
});


Route::get('/books', 'BookController@getIndex');
Route::get('/books/create', 'BookController@getCreate');
Route::post('/books/create', 'BookController@postCreate');
```
