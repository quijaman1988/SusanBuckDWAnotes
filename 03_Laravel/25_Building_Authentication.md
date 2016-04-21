Much of what you need for user authentication comes built into Laravel default. That being said, there is some work you need to do to get things working.

Let's start with what exists...



## Auth Config
Open `config/auth.php` to see the default configurations for how authentication works. For our purposes, we don't have to change anything here, but you should read though the well-commented options.




## User Model
Next, there's the existing `\app\User.php` which is a Eloquent model for interacting with the `users` table. You don't have to change anything here.




## `users` Migration
The default install of Laravel comes with a Migration file to generate the `users` table.

If you open `database/migrations/2014_10_12_000000_create_users_table.php` you can see the structure of this table by examining the `up()` method:

```php
public function up()
{
    Schema::create('users', function (Blueprint $table) {
        $table->increments('id');
        $table->string('name');
        $table->string('email')->unique();
        $table->string('password', 60);
        $table->rememberToken();
        $table->timestamps();
    });
}
```

This migration includes some default fields Laravel will require to have authentication work &ldquo;out of the box&rdquo; including:

1. A `password` field that is at least 60 characters long.
2. A nullable string `remember_token` that's 60 characters long (the `rememberToken()` method creates this field).

You can/should eventually modify this migration to include the fields needed for users in your application.

For example, you may want to add additional fields (e.g. `birth_date`, `postal_code`, `country`, etc.).

Or you may want to edit existing fields like breaking out the `name` field into `first_name` and `last_name`.

For now though, don't make any edits. Let's just get authentication working.

Make sure your migrations have been run and you have the `users` table in your database.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-confirm-users-table-exists@2x.png' style='max-width:1068px; width:100%' alt=''>




## AuthController
Next, open `/app/Http/Controllers/Auth/AuthController.php`. This file comes with Laravel by default and is set up to power all your authentication related routes.

(Despite its sparse appearance, this controller is very powerful because it inherits [AuthenticateUsers.php](https://github.com/laravel/framework/blob/5.2/src/Illuminate/Foundation/Auth/AuthenticatesUsers.php) using [PHP traits](http://php.net/manual/en/language.oop5.traits.php).)

For our purposes, we're only going to make one edit to `AuthController.php` which is to add the following two protected variables right after the `use AuthenticatesAndRegistersUsers, ThrottlesLogins;` line:

```php
# Where should the user be redirected to if their login fails?
protected $loginPath = '/login';

# Where should the user be redirected to after logging out?
protected $redirectAfterLogout = '/';
```

Both of these variables are commented to explain what they do. Without adding these variables, Laravel would use some sensible defaults. For example, if no `$redirectPath` was explicitly set, Laravel would send the user to `/home` by default.




## Routes
Next up for authentication: routes, then views.

There's a Artisan command (`php artisan make:auth`) that will automatically generate the necessary routes and views you'll need, **however, we are not going to use it**. Instead we are going to manually build our own routes and views for the following reasons:

1. So we can understand fully what routes and views are needed
2. So we can make some modifications to the defaults
3. Because we already have a master layout template in place for our views

With that in mind, lets progress forward, adding the following five routes to the `routes.php` file:

```php
# Show login form
Route::get('/login', 'Auth\AuthController@getLogin');

# Process login form
Route::post('/login', 'Auth\AuthController@postLogin');

# Process logout
Route::get('/logout', 'Auth\AuthController@logout');

# Show registration form
Route::get('/register', 'Auth\AuthController@getRegister');

# Process registration form
Route::post('/register', 'Auth\AuthController@postRegister');
```




## Views
By default, the `getLogin()` action will use this view: `resources/views/auth/login.blade.php`.

And the `getRegister()` method will use this view: `resources/views/auth/register.blade.php`

Given that, we need to create these views.

Below is code you can use for `resources/views/auth/login.blade.php`:

```php
@extends('layouts.master')

@section('content')

    <p>Don't have an account? <a href='/register'>Register here...</a></p>

    <h1>Login</h1>

    @if(count($errors) > 0)
        <ul class='errors'>
            @foreach ($errors->all() as $error)
                <li><span class='fa fa-exclamation-circle'></span> {{ $error }}</li>
            @endforeach
        </ul>
    @endif

    <form method='POST' action='/login'>

        {!! csrf_field() !!}

        <div class='form-group'>
            <label for='email'>Email</label>
            <input type='text' name='email' id='email' value='{{ old('email') }}'>
        </div>

        <div class='form-group'>
            <label for='password'>Password</label>
            <input type='password' name='password' id='password' value='{{ old('password') }}'>
        </div>

        <div class='form-group'>
            <input type='checkbox' name='remember' id='remember'>
            <label for='remember' class='checkboxLabel'>Remember me</label>
        </div>

        <button type='submit' class='btn btn-primary'>Login</button>

    </form>
@stop
```

Here are the key points about this particular view:
1. It implements the template inheritance strategy we've been using for all of our foobook's views
2. It's prepared to display errors
3. It makes sure the form has the CSRF token (done with the `csrf_field()` method)
4. It contains the 3 inputs necessary for logging in (`email`, `password`, `remember` checkbox)


Here is code you can use for `resources/views/auth/register.blade.php`:

```php
@extends('layouts.master')

@section('content')

    <p>Already have an account? <a href='/login'>Login here...</a></p>

    <h1>Register</h1>

    @if(count($errors) > 0)
        <ul class='errors'>
            @foreach ($errors->all() as $error)
                <li><span class='fa fa-exclamation-circle'></span> {{ $error }}</li>
            @endforeach
        </ul>
    @endif

    <form method='POST' action='/register'>
        {!! csrf_field() !!}

        <div class='form-group'>
            <label for='name'>Name</label>
            <input type='text' name='name' id='name' value='{{ old('name') }}'>
        </div>

        <div class='form-group'>
            <label for='email'>Email</label>
            <input type='text' name='email' id='email' value='{{ old('email') }}'>
        </div>

        <div class='form-group'>
            <label for='password'>Password</label>
            <input type='password' name='password' id='password'>
        </div>

        <div class='form-group'>
            <label for='password_confirmation'>Confirm Password</label>
            <input type='password' name='password_confirmation' id='password_confirmation'>
        </div>

        <button type='submit' class='btn btn-primary'>Register</button>

    </form>

@stop
```




## Test it: Registration
At this point the following components should be set up:

1. The `users` table
2. The `User` Model
3. Registration, login, and log out routes
4. `AuthController.php`
5. Views for registering and logging in.

Given that, we can test things out.

When you visit `/register` you should see a registration form:

<img src='http://making-the-internet.s3.amazonaws.com/laravel-register-form@2x.png' style='max-width:508px; width:100%' alt='Register form in Laravel'>

If you fill out this form and submit it, you should be logged in and redirected to `http://localhost/` (assuming validation passed).

How do you know it worked? First, check your `users` table and make sure a new row was added.

Second, you can confirm you are logged in by adding and accessing this route:

```php
Route::get('/show-login-status', function() {

    # You may access the authenticated user via the Auth facade
    $user = Auth::user();

    if($user) {
        echo 'You are logged in.';
        dump($user->toArray());
    } else {
        echo 'You are not logged in.';
    }

    return;

});
```

<img src='http://making-the-internet.s3.amazonaws.com/laravel-confirm-logging-in-worked@2x.png' style='max-width:497px; width:100%' alt='Confirmed logging in worked'>




## Test it: Log out
To log a user out, they should visit `http://localhost/logout`.

We'll create a link for this later, but for now you can manually test it by visiting it in the browser (i.e. `http://localhost/logout`).

When you hit the logout URL, it should log you out and redirect you to `http://localhost/` when done.

You can revisit the `/show-login-status` route to confirm the logout work.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-confirm-logging-out-worked@2x.png' style='max-width:497px; width:100%' alt='Confirmed logging in worked'>

## Improving log out
This gets the job done, but could be confusing for the user because there is no confirmation that the login completed.

You could address this in two ways

### Method 1)
Create a `http://localhost/logout/confirm` route that has some text saying &ldquo;*You have been logged out*&rdquo;.

Then, update `$redirectAfterLogout` in `AuthController.php` to point to this route:

```php
# Where should the user be redirected to after logging out?
protected $redirectAfterLogout = '/logout/confirm';
```

### Method 2)
Continue to redirect the user to `http://localhost/` after logging out, but send a flash message to confirm the logout.

In order to do this, you'll need to override the default `logout()` method that is set in [AuthenticateUsers.php](https://github.com/laravel/framework/blob/5.2/src/Illuminate/Foundation/Auth/AuthenticatesUsers.php).

Note, we're *overriding*, not *overwriting*. Overwriting would imply we're changing `AuthenticateUsers.php` which we do not want to do as any changes could be lost in the next update since this is a vendor file.

Instead, we *override* this method my redefining it in our `AuthController.php`:

```php
/**
 * Log the user out of the application.
 *
 * @return \Illuminate\Http\Response
 */
public function logout()
{

    \Auth::logout();

    # This line is our new addition to this method
    \Session::flash('message','You have been logged out.');

    return redirect(property_exists($this, 'redirectAfterLogout') ? $this->redirectAfterLogout : '/');
}
```

Note the addition of the flash message before the direct happens.

Now when you test your `/logout` again, you should be directed to `http://localhost/` with a flash message:
<img src='http://making-the-internet.s3.amazonaws.com/laravel-logout-confirmed@2x.png' style='max-width:853px; width:100%' alt=''>




## Test it: Log in
We know that logging in automatically occurs when the user registers, now let's test and make sure logging in works independently of registration.

Visit `/login` and fill in the credentials you added during the registration test above to confirm logging in is working.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-login-form@2x.png' style='max-width:508px; width:100%' alt='Login form in Laravel'>

If you still have the `/show-login-status` route from above, you can use this again to confirm logging in worked.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-confirm-logging-in-worked@2x.png' style='max-width:648px; width:100%' alt=''>



## Seeds
At this point, you should have a working authentication system complete with login, registration, and logout.

In the next note set, we'll look at examples of utilizing authentication, but before we conclude here, let's set up a Seeder for the `users` table.

Like all the tables in your projects for this course, you want to seed your `users` table with sample data.

In this course, seeding the users table is important/required because it will save us time when helping you debug&mdash; if every student seeds with the same 2 example users (below) we can quickly log into your app without having to register a new user.

First step, create the `UsersTableSeeder`:

```bash
$ php artisan make:seeder UsersTableSeeder
```

In the resulting `/database/seeds/UsersTableSeeder.php` file, update the `run()` method to add two users:
1. Jill (jill@harvard.edu)
2. Jamal (jamal@harvard.edu).

Both these users have the same password, `helloworld` (lowercase, no spaces).

```php
public function run()
{

    $user = \App\User::firstOrCreate(['email' => 'jill@harvard.edu']);
    $user->name = 'Jill';
    $user->email = 'jill@harvard.edu';
    $user->password = \Hash::make('helloworld');
    $user->save();

    $user = \App\User::firstOrCreate(['email' => 'jamal@harvard.edu']);
    $user->name = 'Jamal';
    $user->email = 'jamal@harvard.edu';
    $user->password = \Hash::make('helloworld');
    $user->save();

}
```

Don't forget to also update `/database/seeds/DatabaseSeeder.php` so the `run()` method invokes this new seeder:

```php
$this->call(UsersTableSeeder::class);
```

Test your new seeder to make sure it runs without error.

With this seeding in place, anyone in this course (myself, a TA, a classmate) can log into your site using `jill@harvard.edu` / `helloworld` or `jamal@harvard.edu` / `helloworld`.

Obviously, beyond the scope of this academic setting, should you take your application into the &ldquo;real world&rdquo;, you would want to remove these seeds or update them to use a stronger password.




## Reference
+ [Docs: Authentication](http://laravel.com/docs/authentication)
+ [Docs: Hashing](http://laravel.com/docs/hashing)

Note: Password Resets are an aspect of Authentication that is built into Laravel, but due to time constraints, we're not covering them in this course. If you want to implement password resets in you project, you can refer to the Laravel documentation: <http://laravel.com/docs/authentication#resetting-passwords>
