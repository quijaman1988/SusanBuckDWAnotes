## Background

Just like Laravel supports different database drivers, it also supports different mail drivers.


`/config/mail.php`:

```php
    /*
    |--------------------------------------------------------------------------
    | Mail Driver
    |--------------------------------------------------------------------------
    |
    | Laravel supports both SMTP and PHP's "mail" function as drivers for the
    | sending of e-mail. You may specify which one you're using throughout
    | your application here. By default, Laravel is setup for SMTP mail.
    |
    | Supported: "smtp", "mail", "sendmail", "mailgun", "mandrill", "ses", "log"
    |
    */

    'driver' => env('MAIL_DRIVER', 'smtp'),
```

In these notes, we'll look at using the `SMTP` driver, powered by the transactional email service, [Mailgun](https://mailgun.com). Mailgun offers a free plan which limits your number of outgoing emails, but for the purposes of this class, the limit should not be a problem.




## Set up Mailgun
Create a free account at [Mailgun](http://mailgun.com).

When first getting set up you can use Mailgun's sandbox (i.e. testing) domain.

From your Mailgun control panel (<https://mailgun.com/app/dashboard>) locate the **Your Sandbox Domain** section and note the __Default SMTP Login__ (Username) and __Default Password__.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-mailgun-default-credentials@2x.png' class='' style='max-width:600px; width:100%' alt=''>

This sandbox account is only for testing, so there's a 300 email limit a day. When you're ready to go live with your application, you'd want to authorize your own domain to send emails.




## Configure your app's mail settings
Open `/config/mail.php`.

First, update the `from` settings to match your app.

Example:
```php
'from' => array('address' => 'mail@foobooks.com', 'name' => 'Foobooks'),
```

Next you need to configure `host`, `username`, and `password`. You'll note all 3 of these settings are set using the DOT ENV system:

```php
'host' => env('MAIL_HOST', 'smtp.mailgun.org'),

'username' => env('MAIL_USERNAME'),

'password' => env('MAIL_PASSWORD'),
```

Given this, you should open your local `.env` file and update the following settings:

+ `MAIL_HOST` should be changed from `mailtrap.io` to `smtp.mailgun.org`.
+ `MAIL_USERNAME` should be changed from `null` to the SMTP login you gathered from Mailgun above.
+ `PASSWORD_` should be changed from `null` to the SMTP password you gathered from Mailgun above.


Example after edits:
```
MAIL_DRIVER=smtp
MAIL_HOST=smtp.mailgun.org
MAIL_PORT=2525
MAIL_USERNAME=postmaster@sandbox4783f918183d40879aca024f1c04432e.mailgun.org
MAIL_PASSWORD=a3b1e91f3d2d5d16bf6969115106db37
MAIL_ENCRYPTION=null
```



## Basic example
We now want to do a basic test, sending a plain text email just to confirm your settings are working.

Email is set using the `send` method from the `Mail` facade ([documentation](https://laravel.com/docs/5.2/mail#sending-mail)).

In a practice route/controller set up the following; replace `your@email.com` before testing.

```php
\Mail::send([], [], function ($message) {
  $message->to('your@email.com')
    ->subject('Hello World')
    ->setBody('This is a test message; Testing 123.');
});

return 'Basic, plain text email sent.';
```

Upon hitting the above practice route/controller, assuming there were no errors, you should shortly receive an email from your app.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-test-email-1@2x.png' style='max-width:590px; width:100%' alt=''>

If you do get an error message when you test this code you can confirm your mail settings with the following debug route:

```php
# Just like the debug route we set up when testing DB credentials,
# be sure this code does not end up on a live, production server
# as it reveals sensitive information.
Route::get('/debug-email', function() {
    dump(Config::get('mail'));
});
```

If you do not get an error message, but you also don't get an email, check your spam box.



## Email with a view
Once you've confirmed that email is successfully sending from your application, we can now build a more thorough example using a view. Just like you can structure the display of your web pages with a view, you can also structure the display of HTML emails with a view.

For this email, we're going to imagine a hypothetical feature where users on Foobooks can sign up to put books on their &ldquo;watch list&rdquo;. When books become available, our app sends them an email to let them know.

### Create a view
To start, in your `resources/views/` directory, create a subdirectory called `emails/`; this is where we'll organize any email views.

Within this subdirectory create a blade file called `book-available.blade.php` with the following code:

```php
<h1>Good news {{ $user->name }}!</h1>

<p>The following book is now available on Foobooks.</p>

<div>
    <h2>{{ $book->title }}</h2>
    <h3>By {{ $book->author->first_name }} {{ $book->author->last_name }}</h3>
    <a href='{{ Config::get('app.url').'/book/show/'.$book->id}}'>View now...</a>
</div>

<p>
From,<br>
Team Foobooks
</p>
```

You'll note that there's two main data points we'll need to provide this view:

1. User information
2. Book information


### Program the email
In another practice controller action, set up the following code:

```php
# Get the current logged in user
$user = \Auth::user();

# If user is not logged in, make them log in
if(!$user) return redirect()->guest('login');

# Grab any book, just to use as an example
$book = \App\Book::findOrFail(1);

# Create an array of data, which will be passed/available in the view
$data = array(
    'user' => $user,
    'book' => $book,
);

\Mail::send('emails.book-available', $data, function($message) use ($user,$book) {

    $recipient_email = $user->email;
    $recipient_name  = $user->name;
    $subject  = 'The book '.$book->title.' is now available on Foobooks';

    $message->to($recipient_email, $recipient_name)->subject($subject);

});

echo 'HTML email sent.';
```

Note that the above code requires you to be logged in, and the email will be sent to the email address of the logged in user. Because of this, you can't test this as either Jill or Jamal since you do not actually have access to their made up email accounts (e.g. `jill@harvard.edu`). You'll need to have/use a login for yourself with an email you have access to.

Test your code and confirm you receive the email.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-test-email-2@2x.png' style='max-width:627px; width:100%' alt=''>

Once it's working, study the above code and note that the `send()` method takes three arguments:

1. The name of a view for the email, `emails.book-available`.
2. An array of data to pass the view, `$data`, which holds both `user` and `book`.
3. A closure which receives a $message instance, allowing you to customize the recipients, subject, and other aspects of the mail message. The objects `$user` and `$book` are made available to the closure using the `use` statement because they're needed to fill in the subject and recipient information.





## Mail and your local/testing environments
Once you have real users in your database, you want to be very careful when working on your application that you don't accidentally blast those users with emails.

One way to avoid this is to set your mail driver to `log` on your local/testing environments.

This will pipe out all outgoing emails from your app to `/storage/logs/laravel.log` instead of actually sending the emails. Prevents you from accidentally spamming users in your database!

Example output of email piped to `laravel.log`:

```txt
[2016-04-28 07:33:53] local.DEBUG: Message-ID: <1cb3b74a0878d926e5b30188e2b6884c@foobooks.loc>
Date: Thu, 28 Apr 2016
Subject: The book The Great Gatsby is now available on Foobooks
From: Foobooks <mail@foobooks.com>
To: Susan <dwa15.practice@gmail.com>
MIME-Version: 1.0
Content-Type: text/html; charset=utf-8
Content-Transfer-Encoding: quoted-printable

<h1>Good news Susan!</h1>

<p>The following book is now available on Foobooks.</p>

<div>
    <h2>The Great Gatsby</h2>
    <h3>By F. Scott Fitzgerald</h3>
    <a href='http://foobooks.loc/book/show/1'>View now...</a>
</div>

<p>
From,<br>
Team Foobooks
</p>
```

Alternatively, you can use a service such as [Mailtrap](http://mailtrap.io) which will pipe all outgoing email into a single development inbox (discussed in lecture).




## Reference
+ [Docs: Mail](https://laravel.com/docs/mail)
