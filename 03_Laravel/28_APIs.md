Many sites provide __APIs (Application Program Interfaces)__ allowing external sites to interact with their data.

Examples:

+ [Twitter API](https://dev.twitter.com/rest/public)
    + Read and write Twitter data: Author a new tweet, read author profile and follower data, etc.
+ [Flickr API](https://www.flickr.com/services/api)
    + Read and write Flickr data: Upload photos, get data about a photo, create galleries, etc.
+ [New York Times API](http://developer.nytimes.com)
    + Read NYT data: Article search, book reviews, event listings, movie reviews, etc.

The procedures for tapping into an external API typically follows this pattern:

+ If needed, acquire an API key or other credentials.
+ Find out if there is an existing package/library to help your interface with the API.
+ Find out what methods/params the API accepts, and what kind of data you'll get in return.
+ Implement a basic example as proof of concept that you can interact with the API.
+ Implement a feature on your app that uses the API.

In these notes, I'll go through the above steps using the __[Google Books the API](https://developers.google.com/books/)__.

These steps will work towards building a *&ldquo;Similar books by this author&rdquo;* feature on Foobooks which I'll cover in one of the *Foobooks Progress* topics during lecture.


## Acquire credentials
Most APIs require send identification information with each request so they can track (and sometimes limit) your app's usage of the API.

Credentials for the Google Books API can be created via the [*Google Developer's Console*](https://console.developers.google.com/apis/library); search for `Books API`.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-find-google-books-api@2x.png' style='max-width:656px; width:100%' alt=''>


On the resulting page, find the button to *Enable* the Books API.
<img src='http://making-the-internet.s3.amazonaws.com/laravel-enable-books-api@2x.png' style='max-width:651px; width:100%' alt=''>

Then, go to [__Credentials__](https://console.developers.google.com/apis/credentials) and click __Create Credentials__, then select __API Key__ from the dropdown.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-create-books-api-credentials-key@2x.png' style='max-width:651px; width:100%' alt=''>

On the menu that pops up, choose __Browser key__.
<img src='http://making-the-internet.s3.amazonaws.com/laravel-books-api-choose-browser-key@2x.png' style='max-width:647px; width:100%' alt=''>

Give your Browser key some name (e.g. `Foobooks`).

Leave the `Accept requests from these HTTP referrers` field blank.

Click **Create**.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-books-api-create-browser-key@2x.png' style='max-width:651px; width:100%' alt=''>

Note/Copy the resulting API key, which will be a long strong of numbers, letters, and dashes. We'll need this in a few steps.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-books-api-save-api-key@2x.png' style='max-width:651px; width:100%' alt=''>

## Basic example
The process for interacting with a read-only API is relatively simple:

1. Ping the API URL
2. Parse the results, extracting the data you need (typically the API will respond with data in a format like XML or JSON)

For example, here's an API URL for Google Books:

<https://www.googleapis.com/books/v1/volumes?q=author:maya%20angelou&maxResults=5&key=AIzaSyAuSWKAk2iFAK-uzg-RKCS6L5spPjc46VM>

The URL typically includes both the API **method** you're pinging, and **parameters** for the query.

In this example, the method is `volumes` and the parameters are:

+ `q` (short for query) = `maya%20angelou` (the `%20` is just an encoded space)
+ `maxResults` = `5`
+ `key` = `AIzaSyAuSWKAk2iFAK-uzg-RKCS6L5spPjc46VM`

This URL was constructed by reading this documentation: <https://developers.google.com/books/docs/v1/reference/volumes/list>

If you load the URL in your browser, you'll see the API responds with JSON data:

<img src='http://making-the-internet.s3.amazonaws.com/laravel-books-api-results-in-browser@2x.png' style='max-width:651px; width:100%' alt=''>

You have just interacted with the Google Books API! Now we just need to do that same process from your application.

One way this can be accomplished is using PHP's built in method [file_get_contents()](http://php.net/manual/en/function.file-get-contents.php). Try this out in a practice controller or route:

```php
$apiUrl = "https://www.googleapis.com/books/v1/volumes?q=author:maya%20angelou&langRestrict=en&maxResults=5&key=AIzaSyAuSWKAk2iFAK-uzg-RKCS6L5spPjc46VM";
$jsonStringResults = file_get_contents($apiUrl);
print $jsonStringResults;
```

That should produce something like this:
<img src='http://making-the-internet.s3.amazonaws.com/laravel-books-api-results-in-app@2x.png' style='max-width:651px; width:100%' alt=''>

What you're seeing is the results of `file_get_contents` printed to the browser, i.e. the results of the ping to the Google Books API.

Obviously a large JSON string in this format is not very useful, so the next step is to transform it into an array using PHP's built in method [json_decode()](http://php.net/manual/en/function.json-decode.php).

Update your above example to look like this:

```php
$apiUrl = "https://www.googleapis.com/books/v1/volumes?q=author:maya%20angelou&langRestrict=en&maxResults=5&key=AIzaSyAuSWKAk2iFAK-uzg-RKCS6L5spPjc46VM";
$jsonStringResults = file_get_contents($apiUrl);

$data = json_decode($jsonStringResults, true);

# Show all the data
dump($data);

# Loop through the data printing just the title for each book
foreach($data['items'] as $book) {
    echo $book['volumeInfo']['title'].'<br>';
}
```

That should produce something like this, which gives you an idea of how you can extract the encoded data:

<img src='http://making-the-internet.s3.amazonaws.com/laravel-books-api-results-decoded@2x.png' style='max-width:651px; width:100%' alt=''>

Note how the information we want is nested within the index `volumeInfo`. You'll often have to study API results to get a sense of how the data is structured so you know how to extract out the information you want.

Can you start to imagine how the above code could be used to build a &ldquo;*Similar books by this author*&rdquo; feature on Foobooks?



## Example with an external package
For every API, there are often external code packages/libraries that are developed to make interacting with that API easier.

These packages are typically available for multiple different programming languages, and may be created by the owner of the API itself, or outside developers, or both.

What we're doing with the Google Books API is so simple (as demonstrated by the example above) that an external package is arguably overkill.

However, this isn't always the case, so let's check it out anyway so we have an example...

When trying to find an API package/library, you can look several places:

+ The API documentation
+ Packagist
+ Github/Google

If you find multiple libraries, you'll want to consider the same points we discussed earlier in the semester when looking at [Composer Packages](https://github.com/susanBuck/dwa15-spring2016-notes/blob/master/03_Laravel/12_Composer_Packages.md).

In this case, in the Google Books API documentation, they have a [page listing several libraries for different languages](https://developers.google.com/books/docs/v1/libraries).

Searching for PHP, you'll find the library [api-client-library/php](https://developers.google.com/api-client-library/php/) which leads to this [Github repository](https://github.com/google/google-api-php-client).

<img src='http://making-the-internet.s3.amazonaws.com/laravel-books-api-php-client@2x.png' style='max-width:738px; width:100%' alt=''>

If you were to search on Packagist for &ldquo;Google API&rdquo;, this same package is one of the top hits.
Given this package is built by Google, and is a top hit on Packagist with over 2 million downloads&mdash; it's probably your best bet.




## Using api-client-library/php
To get started, we'll follow [the instructions in the package's README file](https://github.com/google/google-api-php-client) by first installing the package:

```bash
$ composer require google/apiclient:^2.0.0@RC
```

Then, adapt the provided example (also from the package's README file) to meet our needs (see below).

I've added comments to explain what is going on but you should also [read the documentation](https://developers.google.com/api-client-library/php/start/get_started#2-authorized-api-access-oauth-20) under the heading *Building and calling a service*.

Be sure to replace `PASTE-YOUR-API-KEY-HERE` with your own key.



```php
# Instantiate a new object from the library we're using
$client = new \Google_Client();

# Put the name of your application here
$client->setApplicationName("Foobooks");

# Paste in your API Key
$client->setDeveloperKey('PASTE-YOUR-API-KEY-HERE');

# This library can work with multiple different Google APIs, so here we're specifying we're using the Books API
$service = new \Google_Service_Books($client);

# Create an array of params for the query; these are the same params we used in the basic example above
$optParams = [
    'q' => 'author:Maya Angelou',
    'maxResults' => 5,
];

# Each API provides resources and methods, usually in a chain. These can be accessed from the service object in the form $service->resource->method(args)
$books = $service->volumes->listVolumes('', $optParams);

# Output our results to test and make sure it worked
dump($books);

foreach($books['items'] as $book) {
    echo $book['volumeInfo']['title'].'<br>';
}
```

There will be some slight variations in the output, but the data should be the same as when you ran the basic example.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-books-api-php-client-results@2x.png' style='max-width:565px; width:100%' alt=''>


## Move key to config
It's a good idea to extract your API keys into config files so they're centralized and in a logical location.

You can create separate configs for each API you use, or put all your API keys in one `api` config&mdash; it's up to you.

For Foobooks, I'll do the latter...

Create a new file `/config/apis.php` and fill it with this code:

```php
<?php
return [
    'google_api_key' => 'PASTE-YOUR-KEY-HERE'
];
```

Then, when the API key is needed it's used like this:

```php
\Config::get('apis.google_api_key')
```

For example:
```php
$client->setDeveloperKey(\Config::get('apis.google_api_key'));
```



## Writable APIs

In the above example we used *Read* features; we retrieved publicly available information from the Google Books API.

APIs often include *Write* features; for example Twitter has a method where you can author a new Tweet on behalf of a user.

The process of using Write features on an API requires an additional authorization procedure, ensuring that your application has the appropriate privileges to the given user's account.

Many APIs, Google included, use a system called OAuth for this authorization process.

Here's the process, as described in  [this page of the documentation](https://developers.google.com/api-client-library/php/auth/web-app#overview):

>> &ldquo;To use OAuth 2.0 in a web application, first create web application credentials for your project in the Developers Console.

>> Then, when your application needs to access a user's data with a Google API, your application redirects the user to Google's OAuth 2.0 server. The OAuth 2.0 server authenticates the user and obtains consent from the user for your application to access the user's data.

>> Next, Google's OAuth 2.0 server redirects the user back to your application along with a single-use authorization code. Your application exchanges this authorization code for an access token.

>> Finally, your application can use the access token to call Google APIs.&rdquo;

Because of time restrictions, I won't specifically be covering writable APIs in this course. However, if you follow the documentation on your own and get stuck, send me a message and I'd be happy to help you out.
