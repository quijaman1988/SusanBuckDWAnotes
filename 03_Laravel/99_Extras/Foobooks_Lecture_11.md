The following is a rough outline of the modifications I'll make to foobooks during Lecture 11.

This should not be considered a stand-alone document; for full details please refer to the lecture video and the foobooks code source.




## Pre-lecture setup
+ Update `books/create.blade.php` to include `author`, `published`, `cover`, `purchase_link` fields.


## List all the books
`books/index.blade.php` is created, just needs to be filled in.

Use the `Book` model to get all the books.
Pass this data to the View.

```php
public function getIndex() {
    $books = \App\Book::all();
    return view('books.index')->with('books',$books);
}
```

Display the data in the View by looping through it.
```php
@extends('layouts.master')

@section('title')
    All the books
@stop

@section('content')
    <div class='book'>
        @foreach($books as $book)
            <h2>{{ $book->title }}</h2>
            <img src='{{ $book->cover }}'>
        @endforeach
    </div>
@stop
```




## Make /books the homepage as well
```php
Route::get('/', 'BookController@getIndex');
```




## Add a book
`books/create.blade.php` has been updated to include all the fields:

+ author
+ published
+ cover
+ purchase_link

Note the use of `old()` for field value.

Update this so we have some default values to rapidly test with, e.g.

```html
<div class='form-group'>
   <label>* Title:</label>
    <input
        type='text'
        id='title'
        name='title'
        value='{{ old('title','Green Eggs and Ham') }}'
    >
   <div class='error'>{{ $errors->first('title') }}</div>
</div>
```

In the controller, validate the new fields. Title already exists, need to add these:

```php
    'author'        => 'required|min:5',
    'published'     => 'required|min:4',
    'cover'         => 'required|url',
    'purchase_link' => 'required|url',
```

Finish BookController.php `postCreate()` so the book is actually added to the database.

```php
$book = new \App\Book();
$book->title = $request->title;
$book->author = $request->author;
$book->published = $request->published;
$book->cover = $request->cover;
$book->purchase_link = $request->purchase_link;
$book->save();
```

If we keep refreshing the post page, it keeps adding new books. Solve with a redirect.

So far most of our controller methods have been returning Strings or Views.
You can also return a [redirect](http://laravel.com/docs/responses#redirects).

Redirect to a confirmation page...

Or some other logical place like `/books` to view all books.

```php
return redirect('/books');
```

If you do the latter, you will want to [Flash](http://laravel.com/docs/session#flash-data) a confirmation message.

Before redirecting:
```
\Session::flash('flash_message','Your book was added');
```

Explanation of a __flash message__
Explanation of a __Session__
(More on the topic of Sessions when we get to user authentication...)

Then in `master.blade.php`:

```php
@if(Session::get('flash_message') != null))
    <div class='flash_message'>{{ Session::get('flash_message') }}</div>
@endif
```




## Mass Assignment for creating a book

Controller update:
```php
$data = $request->only('title','author','published','cover','cover_link');
$book = new \App\Book($data);
$book->save();

# or:
# \App\Book::create('data')
```

Model update:
```
protected $fillable = ['title', 'author', 'published', 'cover', 'purchase_link'];
```




## Edit a book
Create edit **route** and **controller action**

Create `getEdit($id)` method in the book controller.

Create `books/edit.blade.php` by copying `books/create.blade.php`.

In order to edit a book, we need to know *which* book to edit.

Pass this in as a URL parameter, then hide it in the form as a hidden field.

That way when the form is submitted, the controller action can retrieve the book id to know which book to edit:

```php
$book = \App\Book::find($id);
return view('books.edit')->with('book',$book);
```

Test it with 1,2,3, etc.

Add links from /books.

Handle a bad id:
```php
if(is_null($book)) {
    \Session::flash('flash_message','Book not found');
    return redirect('/books');
}
```

Do the edit:
```php
$book = \App\Book::find($request->id);

$book->title = $request->title;
$book->author = $request->author;
$book->cover = $request->cover;
$book->published = $request->published;
$book->purchase_link = $request->purchase_link;

$book->save();

return 'Book was saved'.
```


Better return:
```php
\Session::flash('flash_message','Your changes were saved');
return redirect('/books/edit/'.$request->id);
```


## Delete a book
Try this one on your own.
