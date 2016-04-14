An Eloquent query returns a __Collection object__.

The Collection object is a wrapper for an Array called **items** which contains all the data of your query *and* methods for working with this data.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-collection@2x.png' class='' style='max-width:951px; width:100%' alt='Eloquent queries return Collections'>





## Collection Magic
Collections come built in with [magic methods](http://php.net/manual/en/language.oop5.magic.php) that lets them (the Collections) adapt to how you use them.

### String magic
For example, if you treat a Collection like a String (e.g. you `echo` it), it will transform itself into a JSON string.

```php
$books = \App\Book::all();

# This will output a JSON string
echo $books;
```

This works because the Collection class contains the [__toString](http://php.net/manual/en/language.oop5.magic.php#object.tostring) magic method which is programmed to output a JSON string.

### Array magic
You can also treat a Collection like an array:

```php
$books = \App\Book::all();

# loop through the Collection and access just the data
foreach($books as $book) {
    echo $book['title']."<br>";
}
```

Depending on your data, the results of the above might look like this:
```text
The Great Gatsby
The Bell Jar
I Know Why the Caged Bird Sings
```

This works because the Collection class implements PHP's [IteratorAggregate](http://php.net/manual/en/class.iteratoraggregate.php) interface.


### Object magic
Or, if you prefer object notation...

```php
$books = \App\Book::all();

foreach($books as $book) {
	echo $book->title."<br>";
}
```

Depending on your data, the results of the above might look like this:
```text
The Great Gatsby
The Bell Jar
I Know Why the Caged Bird Sings
```




## Using Collections in your Views
Because of the above points, you can pass a Collection to a View where it can be iterated through like a regular array or object:

Controller:
```php
function getIndex() {
    $books = \App\Book::all();
    return view('books.index')->with('books', $books);
}
```

View:
```php
@foreach($books as $book)
    <h2>{{ $book->title }}</h2>
@endforeach
```




### Get *just* the data
If you want just a &ldquo;pure&rdquo; array of the data in your Collection, use can use the [toArray()](http://laravel.com/api/5.0/Illuminate/Support/Collection.html#method_toArray) method. This will "strip" out all the methods of the Collection, giving you *just* the data attributes.

```php
$books = \App\Book::all();
dump($books->toArray());
```

Depending on your data, the results of the above might look like this:
```text
array:3 [▼
  0 => array:9 [▼
    "id" => 1
    "created_at" => "2015-11-11 04:03:26"
    "updated_at" => "2015-11-11 04:03:26"
    "title" => "The Great Gatsby"
    "author" => "F. Scott Fitzgerald"
    "page_count" => 0
    "published" => 1925
    "cover" => "http://img2.imagesbn.com/p/9780743273565_p0_v4_s114x166.JPG"
    "purchase_link" => "http://www.barnesandnoble.com/w/the-great-gatsby-francis-scott-fitzgerald/1116668135?ean=9780743273565"
  ]
  1 => array:9 [▶]
  2 => array:9 [▶]
]
```



## Collection Methods
Below is a list of some of the most commonly used methods built into the Collection object.

Refer to the [Collections #Available Methods](https://laravel.com/docs/5.2/collections#available-methods) for a full list.


| Method   |      Usage      |
|----------|-------------|
| [`all()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_all) |  Get all of the items in the collection. |
| [`first()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_first) | Get the first item from the collection. |
| [`last()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_last) | Get the last item from the collection. |
| [`shift()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_shift()) | Get and remove the first item from the collection. |
| [`pop()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_pop) | Get and remove the last item from the collection. |
| [`each()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_each) | Loop through each item in a collection. Can be used as an alternative to a regular `foreach`. |
| [`map()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_map) | Loop through a collection, returning a new collection as a result. Good for copying and editing a collection.
| [`filter()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_filter) | Loop through a collection, returning true/false as you go. What is true stays, what is false gets removed.
| [`sort()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_sort) | Sort through each item with a callback.
| [`reverse()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_reverse) | Reverses a collection.
| [`isEmpty()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_isEmpty) | Determine if the collection is empty or not.
| [`toArray()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_toArray) | Get the collection of items as a plain array.
| [`toJson()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_toJson) | Get the collection of items as JSON.
| [`count()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_count) | Count the number of items in the collection. |

Some of these method names may be familiar, as they also exist as QueryBuilder methods. So what's the difference? Read on...


## Query Responsibility

When building a web application, it's good practice minimize the number of queries you're making to the database, as excess database calls can slow down a site's load time.

One way you can optimize your application is to fetch data from an existing Collection, rather than making another round-trip query on the database. This comes in handy if, for example, you're dealing with a result set in which you want to suss out multiple things from that set: all the books, the first book, the last book, etc. Rather than doing a separate query for each of these needs, you can shift the responsibility over to the Collection and use the data you already fetched...

For example, imagine at the top of a script you call upon the `get()` fetch method to grab all the books for displaying on a table in your View:

```php
$books = \App\Book::orderBy('id','descending')->get();
```
Within the View, you also want to draw attention to the most recently added book to the collection.

To gather this info, you could run this query:

```php
$first = \App\Book::orderBy('id','descending')->first();
```

This would work, but it'd run a new query against the database to find the first book.
The better solution would to fetch the needed info from the *existing* Collection (stored in the `$books` variable).

```php
$first = $books->first();
```

In summary:

```php
/*
2 separate queries on the database:
*/
$books = Book::orderBy('id','descending')->get(); # Query DB
$first_book = Book::orderBy('id','descending')->first(); # Query DB

/*
1 query on the database, 1 query on the collection (better):
*/
$books = Book::orderBy('id','descending')->get(); # Query DB
$first_book = $books->first(); # Query Collection
```



## Read More
+ <https://laravel.com/docs/collections>
+ <http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html>
