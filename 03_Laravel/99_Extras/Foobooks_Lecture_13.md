The following is a very rough outline of the modifications I'll make to foobooks during Lecture 13.

This should not be considered a stand-alone document; for full details please refer to the lecture video and the foobooks code source.





## Google Books
Create new folder `Libraries` in `app/`

Create a new file `GoogleBooks.php`

Run `composer dump-autoload`.

Code:

```php
<?php

namespace App\Libraries;

class GoogleBooks {

    /**
	*
	*/
    public function __construct() {
        $this->client = new \Google_Client();
        $this->client->setApplicationName("Client_Library_Examples");
        $this->client->setDeveloperKey(\Config::get('api.google_client_developer_key'));
    }

    /**
	*
	*/
    public function getOtherBooksByAuthor($author_name, $maxResults = 5) {

        $service = new \Google_Service_Books($this->client);

        $optParams = [
            'q' => 'author:'.$author_name,
            'maxResults' => $maxResults
        ];

        return $service->volumes->listVolumes('', $optParams);
    }

}
```

`BookController.php`, update `getShow()` method:

```php
$googleBooks = new GoogleBooks();
$otherBooksByThisAuthor = $googleBooks->getOtherBooksByAuthor('Maya Angelou');

return view('books.show',[
    'book' => $book,
    'otherBooksByThisAuthor' => $otherBooksByThisAuthor
]);
```


Update view `resources/views/book/show.blade.php`:
```html
<h3>Other books by {{ $book->author->first_name.' '.$book->author->last_name }}</h3>
@foreach($otherBooks as $otherBook)
    <div><a href='{{ $otherBook['volumeInfo']['infoLink'] }}'>{{ $otherBook['volumeInfo']['title'] }}</a></div>
@endforeach
```



## Validating for author not selected

Add a default option to `authorsForDropdown` method in `Author.php`:

```php
public static function authorsForDropdown() {

    [...]
    $authors_for_dropdown = [];
    $authors_for_dropdown[0] = 'Choose an author...';
    [...]
```

Update `BookController::postCreate()` to use the `not_in:0` validation for `author_id`; also add a custom message.

```php
$messages = [
    'not_in' => 'You have to choose an author.',
];

$this->validate($request,[
    'title' => 'required|min:3',
    'published' => 'required|min:4',
    'cover' => 'required|url',
    'purchase_link' => 'required|url',
    'author_id' => 'not_in:0'
],$messages);
```

Make sure the author dropdown has the appropriate error output:

```html
<div class='error'>{{ $errors->first('author_id') }}</div>
```



## Handling bad ids
```php
if(is_null($book)) {
    \Session::flash('message','Book not found');
    return redirect('/books');
}
```



## Making data available to all views with Composers
Goal: Make `user` information available on all views.

Can be done with View Composers: <http://laravel.com/docs/views#view-composers>

First, make the composer:
```bash
$ php artisan make:provider ComposerServiceProvider
```

Then update the resulting composer:
```php
# /app/Providers/ComposerServiceProvider.php
public function boot()
{
    # Make the variable "user" available to all views
    \View::composer('*', function($view) {
        $view->with('user', \Auth::user());
    });
}
```

Register this Service Provider in `config/app.php`:
```php
'providers' => [
    // [...]
    'App\Providers\ComposerServiceProvider'
]
```

Then in any view you can use `$user`, for example:
```html
<li><a href='/logout'>Log out {{ $user->name }}</a></li>
```




## Integrating your existing data and users/auth
Couple different approaches:

1. *One to Many*: Every book is connected to a single user
2. *Many to Many*: Individual books aren't associated with any one user. Instead users can favorite books, powered by a many to many relationship between books and users.

Lets go with #1.



### Connect books and users
To do this, we'll need a migration to update our `books` table to have the `user_id` foreign key. This is the same thing we did when connecting authors and books.

Create the migration:
```bash
$ php artisan make:migration connect_books_and_users
```

Fill in the migration:
```php
public function up()
{
    Schema::table('books', function (Blueprint $table) {

        # Add a new INT field called `user_id` that has to be unsigned (i.e. positive)
        $table->integer('user_id')->unsigned();

        # This field `user_id` is a foreign key that connects to the `id` field in the `authors` table
        $table->foreign('user_id')->references('id')->on('users');

    });
}

public function down()
{
    Schema::table('books', function (Blueprint $table) {

        # ref: http://laravel.com/docs/5.1/migrations#dropping-indexes
        $table->dropForeign('books_user_id_foreign');

        $table->dropColumn('user_id');
    });
}
```



### Update seeders
Because we've created a foreign key between `books` and `authors` our Seeding order needs to be updated; currently it looks like this:

```php
# database/seeds/DatabaseSeeder.php
$this->call(AuthorsTableSeeder::class);
$this->call(BooksTableSeeder::class);
$this->call(TagsTableSeeder::class);
$this->call(BookTagTableSeeder::class);
$this->call(UsersTableSeeder::class);
```

We need to update it so that the `UsersTableSeeder` is invoked *before* the `BooksTableSeeder` (because the `BooksTableSeeder` will have a foreign key connecting to the `users` table).

New order:

```php
$this->call(AuthorsTableSeeder::class);
$this->call(UsersTableSeeder::class); # <-- Happens before books seeder
$this->call(BooksTableSeeder::class);
$this->call(TagsTableSeeder::class);
$this->call(BookTagTableSeeder::class);
```

Finally, update the BooksTableSeeder so that each book is associated with a user.

```php
$author_id = \App\Author::where('last_name','=','Fitzgerald')->pluck('id');
DB::table('books')->insert([
'created_at' => Carbon\Carbon::now()->toDateTimeString(),
'updated_at' => Carbon\Carbon::now()->toDateTimeString(),
'title' => 'The Great Gatsby',
'author_id' => $author_id,
'user_id' => 1, # <--- NEW LINE
'published' => 1925,
'cover' => 'http://img2.imagesbn.com/p/9780743273565_p0_v4_s114x166.JPG',
'purchase_link' => 'http://www.barnesandnoble.com/w/the-great-gatsby-francis-scott-fitzgerald/1116668135?ean=9780743273565',
]);
```

Make the above change for all three books. In our example, we're giving all 3 seeded books to user id 1 (`jill@harvard.edu`).




### Update models
Next, update the `Book` and `User` model so they're aware of this new relationship.

Add this to `Book.php`:
```php
public function user() {
    return $this->belongsTo('\App\User');
}
```

Add this to `User.php`:
```php
public function books() {
    return $this->hasMany('\App\Book');
}
```




### Show the logged in user only their books
Setup complete! Now lets make it so that when a user is logged in they only see *their* books.

Update `BookController.php` `getIndex()` so that the query includes a user_id filter:

```php
public function getIndex(Request $request) {

    # Get all the books "owned" by the current logged in users
    # Sort in descending order by id
    $books = \App\Book::where('user_id','=',\Auth::id())->orderBy('id','DESC')->get();

    return view('books.index')->with('books',$books);
}
```

Test it out.
How many books does Jill see?
Update the `books` table to reduce the number of books that Jill sees and test it again.

How many books does Jamal see? Should be none by default.

Update `/resources/views/books/index.blade.php` to account for this &ldquo;blank slate&rdquo; case:

```php
@if(sizeof($books) == 0)
    You have not added any books.
@else
    @foreach($books as $book)
        <div>
            <h2>{{ $book->title }}</h2>
            <a href='/books/edit/{{$book->id}}'>Edit</a><br>
            <img src='{{ $book->cover }}'>
        </div>
    @endforeach
@endif
```




### Associating books with the logged in user
Update `BookController.php` `postCreate()`:

```php
# Prepare to enter new book into the database
$book = new \App\Book();
$book->title = $request->title;
$book->author_id = $request->author;
$book->user_id = \Auth::id(); # <--- NEW LINE
$book->cover = $request->cover;
$book->published = $request->published;
$book->purchase_link = $request->purchase_link;
```

Once a book is created it is tied to that user; because of this there's no need to do anything with the *Edit Book* in regards to user associations.



### Route adjustments for guests vs. users
Now that books are associated with specific users, it doesn't make sense anymore that guests would be able to see an index of all the books. Instead, guests should see some sort of welcome page.

So, let's create a `WelcomeController` with a `getIndex()` method:

```php
<?php
# /app/Http/Controllers/WelcomeController.php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class WelcomeController extends Controller {

    /**
    * Responds to requests to GET /
    */
    public function getIndex() {

        # Logged in users should not see the welcome page, send them to the books index instead.
        if(\Auth::check()) {
            return redirect('/books');
        }

        return view('welcome.index');
    }

}
```

That `getIndex()` method should load this view:

```php
# /resources/views/welcome/index.blade.php
@extends('layouts.master')

@section('title')
    Welcome to Foobooks
@stop

@section('content')
    <p>
        Welcome to Foobooks, a personal book organizer.
        To get started <a href='/login'>log in</a> or <a href='/register'>register</a>.
    </p>
@stop
```

And then we'll update our routes to look like this:

```php
Route::get('/', 'WelcomeController@getIndex');

Route::group(['middleware' => 'auth'], function() {

    Route::get('/books/create', 'BookController@getCreate');
    Route::post('/books/create', 'BookController@postCreate');

    Route::get('/books/edit/{id?}', 'BookController@getEdit');
    Route::post('/books/edit', 'BookController@postEdit');

    Route::get('/books', 'BookController@getIndex');
    Route::get('/books/show/{title?}', 'BookController@getShow');

});
```

Test it out:
+ While logged out, visit `http://localhost/`&mdash; you should see a welcome page.
+ While logged in, visit `http://localhost/`&mdash; you should see the book listing page.
+ While logged out, try and visit `http://localhost/books`&mdash; you should be directed to the login page.



## Deleting Books
When deleting a book, you have to first delete any relationships to tags (that is, rows in the `book_tag` table).

This can be done via an [Event/Listener](http://laravel.com/docs/5.1/events) but we'll take a shortcut in the interest of time and just embed it in our deletion methods.


Two new routes:
```
Route::get('/books/confirm-delete/{id?}', 'BookController@getConfirmDelete');
Route::get('/books/delete/{id?}', 'BookController@getDoDelete');
```

Two new methods in BookController.php:

```php
public function getConfirmDelete($id) {

    $book = \App\Book::find($id);

    return view('books.delete')->with('book', $book);
}

public function getDoDelete($id) {

    # Get the book to be deleted
    $book = \App\Book::find($id);

    if(is_null($book)) {
        \Session::flash('message','Book not found.');
        return redirect('\books');
    }

    # First remove any tags associated with this book
    if($book->tags()) {
        $book->tags()->detach();
    }

    # Then delete the book
    $book->delete();

    # Done
    \Session::flash('message',$book->title.' was deleted.');
    return redirect('/books');

}
```

One new view:

```php
# resources/books/delete.blade.php
@extends('layouts.master')

@section('title')
    Delete Book
@stop

@section('content')
    <h1>Delete Book</h1>
    <p>Are you sure you want to delete <em>{{$book->title}}</em>?</p>
    <p><a href='/book/delete/{{$book->id}}'>Yes...</a></p>
@stop
```


Add a delete link in `books/index.blade.php`:
```html
<a href='/books/confirm-delete/{{$book->id}}'>Delete</a><br>
```
