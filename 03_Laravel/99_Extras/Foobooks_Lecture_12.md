The following is a very rough outline of the modifications I'll make to foobooks during Lecture 12.

This should not be considered a stand-alone document; for full details please refer to the lecture video and the foobooks code source.



## Book index clean-up
Updated `books.index.blade.php`, putting each book in a section so it can be styled. Also added author info.

```php
@foreach($books as $book)
    <section class='book'>
        <h2>{{ $book->title }}</h2>

        <h3>{{ $book->author->first_name }} {{ $book->author->last_name }}</h3>

        <img src='{{ $book->cover }}' alt='Cover for {{$book->title}}'>

        <br><a href='/book/edit/{{$book->id}}'>Edit</a>
    </section>
@endforeach
```

Need to eager load the authors:

```php
$books = \App\Book::with('author')->orderBy('id','desc')->get();
```



Added `books_index.css`:
```css
.book {
    border:1px solid #ccc;
    margin-right:10px;
    margin-bottom:10px;
    min-width:300px;
    width:25%;
    float:left;
    padding:10px;
}

.book h2 {
    font-size:1.2em;
    margin:1px;
}

.book h3 {
    font-size:1em;
    margin:1px;
}
```

Linked it at the top of `books/index.blade.php`:

```php
@section('head')
    <link href='/css/books_index.css' rel='stylesheet'>
@stop
```



## Edit Book + Authors
Previously when `author` was just a text field in the `books` table, we just needed a text input to update the author:

```html
<label for='title'>* Author:</label>
<input
    type='text'
    id='author'
    name="author"
    value='{{$book->author}}'
>
```

Now that we've extracted authors to their own table, we need to switch our approach from a text input to a dropdown filled with all the authors.

To make this happen, we first need an __array of authors__ where the key is the author `id` and the value is the author `name`.

```php
# BookController.php
public function getEdit($id = null) {

    $book = \App\Book::find($id);

    # Get all the authors
    $authors = \App\Author::orderBy('last_name','ASC')->get();

    # Organize the authors into an array where the key = author id and value = author name
    $authors_for_dropdown = [];
    foreach($authors as $author) {
        $authors_for_dropdown[$author->id] = $author->last_name.', '.$author->first_name;
    }

    [...]

    # Make sure $authors_for_dropdown is passed to the view
    return view('books.edit')->with([
        'book' => $book,
        'authors_for_dropdown' => $authors_for_dropdown
    ]);
```

Then, we can construct the dropdown (`<select>`) using this data:

```html
<label for='author_id'>* Author:</label>
<select id='author_id' name='author_id'>
    @foreach($authors_for_dropdown as $author_id => $author_name)
         <option value='{{$author_id}}' {{ ($book->author_id == $author_id) ? 'SELECTED' : '' }}>
             {{$author_name}}
         </option>
     @endforeach
</select>
```


Now in `postEdit()` set the `author_id` using the data from the dropdown in `$request`:
```php
$book->author_id = $request->author_id;
```

Test it out to make sure it's working.



## Custom Model Methods
We'll want to do the same thing on the *Add a Book* page.

Rather than duplicate the &ldquo;get authors for dropdown&rdquo; code, we should extract it out of the controller and add it as a method to the `Author` model:

```php
# app/Author.php
public static function authorsForDropdown() {

    $authors = \App\Author::orderBy('last_name','ASC')->get();
    $authors_for_dropdown = [];
    foreach($authors as $author) {
        $authors_for_dropdown[$author->id] = $author->last_name.', '.$author->first_name;
    }

    return $authors_for_dropdown;
}
```

Then in `getEdit()`:
```
$authors_for_dropdown = \App\Author::authorsForDropdown();
```

__Discussion:__ &ldquo;Skinny controllers, fat models&rdquo;

Other frequently used queries can also be abstracted, for example:

```php
public static function getAllBooksWithAuthors() {
    return \App\Book::with('author')->orderBy('id','desc')->get();
}
```

## On your own
Apply the same procedures covered above to switch the *Add a Book* page from a author text input to an author drop down.



## Using Tags/Many to Many
Last lecture, we set everything up (migrations, seeders, models) for a tags feature, now lets look at how we'd implement tags.

We need a way to associate tags with books (either from the *Edit Book* or *Create Book* page)

For authors, this was done with a dropdown which worked because each book can have only *one* author.

Each book can have *many* tags, though, so a dropdown won't do. Instead, lets show all possible tags with checkboxes.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-foobooks-tag-checkboxes@2x.png' style='max-width:357px; width:100%' alt='Tags checkboxes'>

To accomplish this, we'll need to gather the following data:

1. All the possible tags
2. All the tags associated with the book we're looking at.


First, a `getTagsForCheckboxes()` method in the Tag model:

```php
# Tag.php
public function getTagsForCheckboxes() {

    $tags = \App\Tag::orderBy('name','ASC')->get();

    $tagsForCheckboxes = [];

    foreach($tags as $tag) {
        $tagsForCheckboxes[$tag['id']] = $tag;
    }

    return $tagsForCheckboxes;

}
```

Then update `BookController.php` `getEdit()`:

```php
# BookController.php
public function getEdit($id = null) {

    # Get this book and eager load its tags
    $book = \App\Book::with('tags')->find($id);

    [...]

    # Get all the possible tags so we can include them with checkboxes in the view
    $tags_for_checkbox = \App\Tag()::getTagsForCheckboxes();

    /*
    Create a simple array of just the tag names for tags associated with this book;
    will be used in the view to decide which tags should be checked off
    */
    $tags_for_this_book = [];
    foreach($book->tags as $tag) {
        $tags_for_this_book[] = $tag->name;
    }
    # Results in an array like this: $tags_for_this_book['novel','fiction','classic'];

    return view('books.edit')
        ->with([
            'book' => $book,
            'authors_for_dropdown' => $authors_for_dropdown,
            'tags_for_checkbox' => $tags_for_checkbox,
            'tags_for_this_book' => $tags_for_this_book,
        ]);

}
```

```php
# /resources/views/books/edit.blade.php

[...]

@foreach($tags_for_checkboxes as $tag_id => $tag_name)
    <input
        type='checkbox'
        value='{{ $tag_id }}'
        name='tags[]'
        {{ (in_array($tag_id,$tags_for_this_book)) ? 'CHECKED' : '' }}
    >
    {{ $tag_name }} <br>
@endforeach

[...]
```


```php
# BookController.php
public function postEdit(Request $request) {

    [...]

    $book->save();

    # If there were tags selected...
    if($request->tags) {
        $tags = $request->tags;
    }
    # If there were no tags selected (i.e. no tags in the request)
    # default to an empty array of tags
    else {
        $tags = [];
    }

    [...]

}
```

Test it out by removing and adding tags.
