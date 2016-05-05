To understand ajax, lets first look at how we've been processing forms up until this point:

In some view we'd have a form like this:

```html
<form method='POST' action='/book/update'>

	[...Input fields for book data like
	title, author, published date, etc...]

	<input type='submit'>
</form>
```

With a form like this, when the user hit the submit button, it would submit the form, directing the user to `/book/update`, including the data from the form.

That page (which in our case is a route/controller action) then handled the data and responded somehow, whether it be to display a confirmation message, redirect the user somewhere else, etc.

The key part of the above interaction is that to submit the form data, you submitted a new page request to the server. The benefit of ajax is avoiding this page request.

Imagine doing the following actions...

* Posting a comment on your friend's Facebook wall
* Adding an item to a shopping cart from an online store
* Dragging around a Google Map in order to see a new portion of that map

Thinking about these interactions, you'll note that the page does not refresh each time you trigger the action (submit comment, add to cart, drag map).

Instead, data is sent to the server &ldquo;behind the scenes&rdquo;, the server returns a response, and the existing page is updated to let you know your action was successful.

This is ajax&mdash; __a technique that allows a site to communicate with its server (and the server to respond), even after the page has completed loading__. This means your user can invoke small actions throughout your page and get near-immediate feedback with no page load required.



## Technical definition
Ajax stands for **Asynchronous JavaScript** and **XML**. It's not one single technology but actually a combination of technologies.

Let's break down the different pieces:

**Asynchronous** describes the &ldquo;behind the scenes&rdquo; nature of ajax. Data is transferred from the browser to the server, all behind the scenes without interrupting other requests on the page. This is in contrast with *synchronous*, where the browser is effectively frozen while a request is being processed.

**JavaScript** is in charge of sending the data from the browser to the server, commonly using a browser API called the [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)). In addition to sending the request, JS is in charge of receiving and processing any data returned from the server.

**XML** stands for *Extensible Markup Language* and it's describing the format of the data that's sent back from the server. XML is similar to HTML in that it's a tag based language, but it's made specifically for transferring data between machines. All that being said, you don't have to use XML as your data protocol. It's common to have the server respond with JSON, HTML or other formats.



## Example
<img src='http://making-the-internet.s3.amazonaws.com/laravel-ajax-search-for-books.gif'>

Demo:

+ <http://foobooks.dwa15-practice.biz/book/search>
+ Login with `jill@harvard.edu` / `helloworld`

Here's a rundown of the essential pieces of how this is working:

Two routes:
```
Route::get('/book/search', 'BookController@getSearch');
Route::post('/book/search', 'BookController@postSearch');
```

The first route (GET) is to show a search form.

The second route (POST) will respond to the ajax call requesting books.


## GET /search/book - Show form to search for book
For the GET route, set up the BookController@getSearch method so that it returns a view called `search.blade.php` with the following contents:

```php
@extends('layouts.master')

@section('title')
    Search for a book
@stop

@section('head')
    <link href='/css/search.css' rel='stylesheet'>
@stop

@section('content')

    <h1>Search for a book</h1>

    <form method='POST'>

        {{ csrf_field() }}

        <div class='form-group'>
           <label>Search for a book title, e.g. 'Gatsby' or 'Caged Bird':</label>
            <input
                type='text'
                id='search'
                name='search'
            >
            <i id='loading' class="fa fa-refresh fa-spin"></i>
        </div>

        <h2>Results:</h2>
        <div id='results'></div>

    </form>

@stop

@section('body')
    <script src="/js/search.js"></script>
@stop
```

Note the following points about the above view:
+ There's a form with a csrf token, and a text input for the search term.
+ There's an empty div with the id #results; this is where results from the ajax call will be written to.
+ A file `/js/search.js` is included at the bottom. The JS to handle the ajax call will go in that file; we'll come back to it in a moment.


## POST /search/book - Responds to ajax request for books
For the POST route, it's set up to respond to a POST request that sends a search term (`search`). It uses this value to perform a query on the books table and returns a view called `books.search-ajax`.

That code looks like this:

```php
/**
* Responds to requests to POST /book/search/
* This method is used in response to an ajax request from GET /book/search
* See /public/js/search.js
*/
public function postSearch(Request $request) {
    # Do the search with the provided search term ($request->search)
    $books = \App\Book::where('title','LIKE','%'.$request->search.'%')->get();
    # Return the view with the books
    return view('books.search-ajax')->with(
        ['books' => $books]
    );
}
```

You'll note the above method returns the view `books.search-ajax`, so the next step is to create that.

Create `/resources/views/books/search-ajax.blade.php` with this code:

```html
@if(sizeof($books) == 0)
    No results found.
@endif

@foreach($books as $book)
    <div class='book'>
        <a href='/book/show/{{$book->id}}'>{{ $book->title }}</a>
    </div>
@endforeach
```

Note how this view loops through `$books`, which it'll receive from the controller.

Also note how it's a fragment template&mdash; it does not extend the master layout. That's because this view is not intended to be used in isolation, but as an ajax response that gets injected into some existing page.



## The JavaScript
At this point you have a GET route to show the form, and a POST route to respond to requests from the form, now we just need JavaScript to make those requests.

Create `/public/js/search.js` and fill it with this code:

```js
// The following anonymous function will trigger every time the key goes up
// in the search input; i.e. someone changes the search input.
$('#search').keyup(function() {

    // Prevents the submit button from trying to submit the form;
    // This is important because we don't want to traditionally submit the form,
    // Instead we want to manually submit it with ajax.
    event.preventDefault();

    // Set up the ajax call; see http://api.jquery.com/jquery.ajax for more details
    $.ajax({
        url: '/book/search', // Route that will handle the request; its job is to return us books.
        method: 'POST',
        dataType : 'html', // Kind of data we're expecting to get back
        data: { // Two pieces of data we'll send with the request
            '_token': $('input[name=_token]').val(),
            'search': $('#search').val()
        },
        // What to do before each ajax
        beforeSend: function() {
            $('#loading').show();
            $('#results').removeClass('error');
        },
        // What to do upon completion of a successful ajax call
        success: function(data) {
            $('#loading').hide();
            $('#results').html(data);
        },
        // What to do upon completion of an unsuccessful ajax call
        error: function() {
            $('#results').html('Sorry, there was an error; your request could not be completed.');
            $('#results').addClass('error');
        }

    });

});
```

In order for this code to work, your page needs to include jQuery. In foobooks, jQuery is included via the [master template](https://github.com/susanBuck/foobooks/blob/master/resources/views/layouts/master.blade.php#L63).

Read through the JavaScript for `search.js`, as it's commented to explain what is going on. If you're unfamiliar with JavaScript, you might find this code challenging to understand. Feel free to ask myself or a TA for further explanation as needed.


## Summary
Those are all the pieces needed to make this work... Now lets run through what happens:

+ You visit `http://localhost/book/search` and see a search form.
+ You start typing into the search box, which triggers the `keyup` event listener in your `search.js` file.
+ The event listener executes the ajax call, which sends a request via POST to `http://localhost/book/search`. The request includes the search term.
+ `postSearch()` receives the request and queries the Book model with the given search term.
+ Resulting books are passed to a view, which is then returned.
+ Now the ajax call receives this response, and injects it into the `#results` div on the page.
