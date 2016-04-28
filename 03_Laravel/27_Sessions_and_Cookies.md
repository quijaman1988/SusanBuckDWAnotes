Working with Laravel's authentication system is a bit of a black box. With a few simple method invocations you can build a fully functioning login system, without really understanding how it's working.

Given this, let's open the box and take a closer look at how the system works, specifically the relationship between __Cookies__, __Sessions__ and the `users` table in the database.

Before we jump in, it's important that you've already followed the notes on [Authentication]() before continuing here.

Aside from that... some starting facts:

**Cookies** are small text payloads made up of a `key,value` pair that are stored on a user's browser.

A site can create cookies, and retrieve cookies on the __user's browser__.

Cookies are domain specific. This means a site can only access Cookies that it created.

**Sessions** are also small text payloads made up of `key,value` pairs, but instead of being stored on the user's browser, they're stored on the __site's server__.

Sessions can be stored in a variety of ways on the server, but in our examples, we're using __file based__ sessions. This means all sessions live in the directory `/app/storage/sessions` within your Laravel project.

Regardless of where sessions are stored, Laravel abstracts our interaction with them, so the usage is the same.

Cookies and sessions are used in combination to recall information about your users as they visit your site.




## Useful output
Before digging into how cookies and sessions work together, here is some code that will output debugging information that is help helpful when dissecting what is going on.

Set this code up in a route or controller so you can follow along:
<https://gist.github.com/susanBuck/fe62ea527c8b2cf9c579>




## New visitor, not logged in
In our first scenario, let's dissect cookies and sessions for the perspective of a new user on your application, who is not logged in.

This new user goes to your home page for the first time. There's nothing on your homepage that requires a session or cookie, so none are created.

Then this user navigates to your *Add a book* page. This page should have a csrf token which is a feature that depends on cookies/sessions. Given this, your app will see if a cookie/session already exists, and if not, it will add a new browser cookie called `laravel_session`.

At this point, if you examine your cookies (assuming you're acting as the &ldquo;new user&rdquo;) via the Web Inspector, you can see this cookie:

<img src='http://making-the-internet.s3.amazonaws.com/laravel-session-cookie@2x.png' class='' style='max-width:915px; width:100%' alt=''>

You can also output the cookie like this:
```php
echo var_dump($_COOKIE['laravel_session']);
```

In our example, this is what the cookie looks like:

```txt
 [laravel_session] => string(276) "eyJpdiI6IkhVSzZyOVpsUzFkQUtRTXhPdUZpN1E9PSIsInZhbHVlIjoiWkJJbG02YzhhUGlldStleXpOWHg0VEdOWDMxdTFIK0JoVEMxVCtwbUpMK3Joa2Y2QllZWUtoZGNEOE45ZnlpNlQwb1pWd3RqMnRuZ1U5aElqMk5HaWc9PSIsIm1hYyI6ImVmZDA5ZDg5YjRiYTI2MTE0Y2NlYjFjZWExMWRkYjcwODllY2Q3OWU2ZjY3MWQ5N2RiNjg3NTEzZWU2MWQ1OTYifQ=="
```

All cookies created by the Laravel framework are encrypted and signed with an authentication code, meaning they will be considered invalid if they have been changed by the client ([ref](http://laravel.com/docs/requests#cookies)).

Given this, the data we're seeing for `laravel_session` is actually an encrypted value. If you want to see the *unencrypted* version, you can use Laravel's cookie helper:

```php
echo var_dump(Cookie::get());
```

Output:
```
[laravel_session] => string(40) "a5145831c801a8caa6ec77ac4a58cfb61392010b"
```

This unencrypted string corresponds to the name of your session file&mdash; It's how Laravel can make the connection between your browser cookie and the server session.

Given this, if you look in `/storage/framework/sessions` (where session files are stored), you should see a file called `a5145831c801a8caa6ec77ac4a58cfb61392010b`.

The contents of that file will look something like this:

```txt
: a:4:{s:6:"_token";s:40:"aeFImfzRz3Ptr2o9O49K9MHZlBxQF2SVBHwiNXZ5";s:5:"flash";a:2:{s:3:"new";a:0:{}s:3:"old";a:0:{}}s:22:"PHPDEBUGBAR_STACK_DATA";a:0:{}s:9:"_sf2_meta";a:3:{s:1:"u";i:1416382453;s:1:"c";i:1416381868;s:1:"l";s:1:"0";}}
```

In the above example the important part is the key `_token` that is set to the value `aeFImfzRz3Ptr2o9O49K9MHZlBxQF2SVBHwiNXZ5`. **This value is your session token**.

This session token is used, among other places, for your CSRF tokens.

To recap:
If a user visits a Laravel site...

1. It looks for the `laravel_session` cookie in the user's browser. If this cookies doesn't exist, your app will create it.
2. Your app uses the value of this cookie to look up the user's session on the server.
3. Within the session is a unique session token that identifies the user to the site.




## Logged in user
When you log in to a Laravel app, your `laravel_session` cookie changes, pointing to a new session on the server.

Here's our session after logging in:

Contents of `/storage/framework/sessions/02d7477afb8929faea84e83ace5684d093aa82f0`:
```txt
a:5:{s:6:"_token";s:40:"aeFImfzRz3Ptr2o9O49K9MHZlBxQF2SVBHwiNXZ5";s:5:"flash";a:2:{s:3:"new";a:0:{}s:3:"old";a:0:{}}s:38:"login_82e5d2c56bdd0811318f0cf078b78bfc";s:1:"1";s:22:"PHPDEBUGBAR_STACK_DATA";a:0:{}s:9:"_sf2_meta";a:3:{s:1:"u";i:1416383871;s:1:"c";i:1416381868;s:1:"l";s:1:"0";}}
```

Note the key `_token` is still set to the value `aeFImfzRz3Ptr2o9O49K9MHZlBxQF2SVBHwiNXZ5`.

But there's also a new key `login_82e5d2c56bdd0811318f0cf078b78bfc` which is set to the value `"1"` which corresponds to the `id` of the user in the `users` table.

To recap, if you're logged into a Laravel site and you come back to that site here's how it &ldquo;remembers&rdquo; you:

1. It looks for the `laravel_session` cookie in your browser.
2. It uses the value of this cookie to look up your session on the server.
3. Within the session, it can find your user `id` and look up your corresponding row in the `users` table.




## remember_token
When authenticating a user, the `attempt()` method accepts a second parameter called `$remember` which you can set to true or false:

```php
Auth::attempt($credentials, $remember = true)
```

If you set `$remember` to `true`, a Cookie called `remember_82e5d2c56bdd0811318f0cf078b78bfc` is generated when the user logs in (this is in addition to the `login_` cookie).

This remember Cookie contains your user id *and* a unique string that maps to your `remember_token` field in the database.

Ex:
```
[remember_82e5d2c56bdd0811318f0cf078b78bfc]
=> string(62) "1|wkt3YLrHC5HEcIXPkeqxpMt1s7Y9yDYUZrMXySKdfYEk0q0YpjK05j3ypeHM"
```

The remember token is refreshed on login/logout and is used to prevent against &ldquo;remember me&rdquo; cookie hijacking. This is an exploit where a hacker may steal your Cookie info, put it on their system, and attempt to access the app as if they are you, logged in. By refreshing the remember token on logout, it would invalidate any such attacks.

You'll note that, by default, even if you set `$remember` to `false`, it will still remember you if you close and reopen your browser.

This is because the app is looking you up via the Session, not the remember Cookie.

To disable this, you'd want to edit `/app/config/session.php` so that `expire_on_close` is set to true:

```
'expire_on_close' => true,
```

Here are some example combinations showing how the remember token and the Session `expire_on_close` setting interact with each other:

+ `$remember = false`, `'expire_on_close' => false`, User will be remembered because even though there's no remember_token, the Session will not expire on close.

+ `$remember = false`, `'expire_on_close' => true`, User will **not** be remembered because there's no remember_token *and* the Session will expire on close.

+ `$remember = true`, `'expire_on_close' => true`, User will be remembered because even though the Session will expire on close, there's a remember_token.




## Misc

__Key names__

Regarding the login key `login_82e5d2c56bdd0811318f0cf078b78bfc`: this name is not unique for each user. It's a combination of `login_'.md5('Illuminate\Auth\Guard');`

Same thing for `remember_82e5d2c56bdd0811318f0cf078b78bfc`.

See `vendor/laravel/framework/src/Illuminate/Auth.php` for more details.

__Default Cookie name__

By default the Cookie name Laravel uses is `laravel_session`, but you can overwrite this in `/app/config/session.php`.
