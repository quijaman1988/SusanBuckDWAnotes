## Database Relationships
A web application typically contains multiple database tables, and those tables often relate to one another in some way.

There are 6 different relationship types:

+ One To One
+ One To Many
+ Many To Many
+ Has Many Through
+ Polymorphic Relations
+ Many To Many Polymorphic Relations

You can read about all the relationships here: [Docs: Eloquent Relationships](http://laravel.com/docs/eloquent-relationships).

In these notes, we'll start with the __One to Many__ relationship.




## Authors
Continuing with the Foobooks example, we can expand on our database design to include an `authors` table.

This addition introduces the following relationship to our database:

__[One to Many](http://laravel.com/docs/eloquent#one-to-many)__

+ Author *has many* Books
+ Inverse: Books *belongs to* Author

In the following steps we'll set up everything this relationship needs to work.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-one-to-many@2x.png' style='max-width:1020px; width:100%' alt='One To Many Relationship'>


## Create the authors table
First we need to create, build, and run the migration to create a new `authors` table.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-authors-table@2x.png' style='max-width:452px; width:100%' alt=''>

Create the migration:
```bash
$ php artisan make:migration create_authors_table
```

The migration should look like this:
```php
<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateAuthorsTable extends Migration
{

    public function up()
    {
        Schema::create('authors', function (Blueprint $table) {

            $table->increments('id');
            $table->timestamps();

            # The rest of the fields...
            $table->string('first_name');
            $table->string('last_name');
            $table->integer('birth_year');
            $table->string('bio_url');

        });
    }

    public function down()
    {
        Schema::drop('authors');
    }
}
```


Since we've created an `authors` table, we should also generate an `Author` model:
```bash
$ php artisan make:model Author
```



## Seed the authors table

Create the seeder:
```bash
$ php artisan make:seeder AuthorsTableSeeder
```

The seeder should look like this:
```php
<?php

use Illuminate\Database\Seeder;

class AuthorsTableSeeder extends Seeder
{

    public function run()
    {
        DB::table('authors')->insert([
        'created_at' => Carbon\Carbon::now()->toDateTimeString(),
        'updated_at' => Carbon\Carbon::now()->toDateTimeString(),
        'first_name' => 'F. Scott',
        'last_name' => 'Fitzgerald',
        'birth_year' => 1896,
        'bio_url' => 'https://en.wikipedia.org/wiki/F._Scott_Fitzgerald',
        ]);

        DB::table('authors')->insert([
        'created_at' => Carbon\Carbon::now()->toDateTimeString(),
        'updated_at' => Carbon\Carbon::now()->toDateTimeString(),
        'first_name' => 'Sylvia',
        'last_name' => 'Plath',
        'birth_year' => 1932,
        'bio_url' => 'https://en.wikipedia.org/wiki/Sylvia_Plath',
        ]);

        DB::table('authors')->insert([
        'created_at' => Carbon\Carbon::now()->toDateTimeString(),
        'updated_at' => Carbon\Carbon::now()->toDateTimeString(),
        'first_name' => 'Maya',
        'last_name' => 'Angelou',
        'birth_year' => 1928,
        'bio_url' => 'https://en.wikipedia.org/wiki/Maya_Angelou',
        ]);

    }
}
```

Update the `run()` method in `/database/seeds/DatabaseSeeder.php` to invoke this new seeder:

```php
public function run()
{
	Model::unguard();

	/*
	Because Books will be associated with Authors,
	we need to seed Authors first
	*/
	$this->call(AuthorsTableSeeder::class);
	$this->call(BooksTableSeeder::class);

	Model::reguard();
}
```




## Connect authors and books tables
Now that we have an `authors` table, we need to modify the `books` table adding a foreign key field that will connect each book to a row in the `authors` table.

Create the migration:
```bash
$ php artisan make:migration connect_authors_and_books
```

The migration should look like the following.

Note that foreign keys have to be __unsigned__ (i.e. positive).

```php
<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class ConnectAuthorsAndBooks extends Migration
{
    public function up()
    {
        Schema::table('books', function (Blueprint $table) {

			# Add a new INT field called `author_id` that has to be unsigned (i.e. positive)
            $table->integer('author_id')->unsigned();

			# This field `author_id` is a foreign key that connects to the `id` field in the `authors` table
            $table->foreign('author_id')->references('id')->on('authors');

        });
    }

    public function down()
    {
        Schema::table('books', function (Blueprint $table) {

            # ref: http://laravel.com/docs/migrations#dropping-indexes
            # combine tablename + fk field name + the word "foreign"
            $table->dropForeign('books_author_id_foreign');

            $table->dropColumn('author_id');
        });
    }
}
```




## Update the books table seeder
Now that we have an authors table and seeder, we should modify the existing `BooksTableSeeder` to connect each book to the appropriate author:

```php
public function run()
{

    $author_id = \App\Author::where('last_name','=','Fitzgerald')->pluck('id')->first();
    DB::table('books')->insert([
    'created_at' => Carbon\Carbon::now()->toDateTimeString(),
    'updated_at' => Carbon\Carbon::now()->toDateTimeString(),
    'title' => 'The Great Gatsby',
    'author_id' => $author_id,
    'published' => 1925,
    'cover' => 'http://img2.imagesbn.com/p/9780743273565_p0_v4_s114x166.JPG',
    'purchase_link' => 'http://www.barnesandnoble.com/w/the-great-gatsby-francis-scott-fitzgerald/1116668135?ean=9780743273565',
    ]);

    $author_id = \App\Author::where('last_name','=','Plath')->pluck('id')->first();
    DB::table('books')->insert([
    'created_at' => Carbon\Carbon::now()->toDateTimeString(),
    'updated_at' => Carbon\Carbon::now()->toDateTimeString(),
    'title' => 'The Bell Jar',
    'author_id' => $author_id,
    'published' => 1963,
    'cover' => 'http://img1.imagesbn.com/p/9780061148514_p0_v2_s114x166.JPG',
    'purchase_link' => 'http://www.barnesandnoble.com/w/bell-jar-sylvia-plath/1100550703?ean=9780061148514',
    ]);

    $author_id = \App\Author::where('last_name','=','Angelou')->pluck('id')->first();
    DB::table('books')->insert([
    'created_at' => Carbon\Carbon::now()->toDateTimeString(),
    'updated_at' => Carbon\Carbon::now()->toDateTimeString(),
    'title' => 'I Know Why the Caged Bird Sings',
    'author_id' => $author_id,
    'published' => 1969,
    'cover' => 'http://img1.imagesbn.com/p/9780345514400_p0_v1_s114x166.JPG',
    'purchase_link' => 'http://www.barnesandnoble.com/w/i-know-why-the-caged-bird-sings-maya-angelou/1100392955?ean=9780345514400',
    ]);
}
```




## Migrate and Seed
If the above steps were executed correctly, you should be able to run the following command to rebuild and seed your tables:

```bash
$ php artisan migrate:refresh --seed
```

If you have any issues with the above steps, revisit the notes on Migrations and Seeding.

&mdash;

This concludes our table setup for this relationship. Next, we'll look at how we can tell our Models about these relationships.




## Identifying Relationships in Models
Relationships between tables need to be defined in their corresponding Models via __relationship methods__.

For example, the Author model should have a method called `book()` which returns the Eloquent relationship [hasMany](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Model.html#method_hasMany):


```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Author extends Model

	public function book() {
		# Author has many Books
		# Define a one-to-many relationship.
		return $this->hasMany('\App\Book');
	}
}
```

On the flip side, the Book class should have a method called `author()` which returns the Eloquent relationship [belongsTo](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Model.html#method_belongsTo):

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Book extends Model {

	public function author() {
		# Book belongs to Author
		# Define an inverse one-to-many relationship.
		return $this->belongsTo('\App\Author');
	}
}
```

Now that we've made these Models &ldquo;aware&rdquo; of these relationships, we can start using them...




## Associate an author with a book
Here's an example where we create a book, and then associate that book with an author.

To do this, we'll first create a new author:

```php
$author = new \App\Author;
$author->first_name = 'J.K';
$author->last_name = 'Rowling';
$author->bio_url = 'https://en.wikipedia.org/wiki/J._K._Rowling';
$author->birth_year = '1965';
$author->save();
dump($author->toArray());
```

Then we'll create a new book and associate it with the author:

```php
$book = new \App\Book;
$book->title = "Harry Potter and the Philosopher's Stone";
$book->published = 1997;
$book->cover = 'http://prodimage.images-bn.com/pimages/9781582348254_p0_v1_s118x184.jpg';
$book->purchase_link = 'http://www.barnesandnoble.com/w/harrius-potter-et-philosophi-lapis-j-k-rowling/1102662272?ean=9781582348254';
$book->author()->associate($author); # <--- Associate the author with this book
$book->save();
dump($book->toArray());
```

Note how the `associate()` method is called *before* the `save()` method. This is because `associate` is setting the `author_id` field on the books row; that setting should happen before the row is created.

Another way to look at the `associate()` method is that it's doing this:

```php
$book->author_id = $author->id;
```





## Querying with relationships
Once your Models have been programmed with relationships, it's easy join data amongst multiple tables using [dynamic properties](http://laravel.com/docs/eloquent#dynamic-properties).

>> Eloquent allows you to access your relations via *dynamic properties*. Eloquent will automatically load the relationship for you, and is even smart enough to know whether to call the get (for one-to-many relationships) or first (for one-to-one relationships) method. It will then be accessible via a dynamic property by the same name as the relation.

For example, you can fetch a book and have access to its author info...

```php
# Get the first book as an example
$book = \App\Book::first();

# Get the author from this book using the "author" dynamic property
# "author" corresponds to the the relationship method defined in the Book model
$author = $book->author;

# Output
echo $book->title.' was written by '.$author->first_name.' '.$author->last_name;
dump($book->toArray());
```



## Eager Loading
If you're querying for many books, you may want to join in the related author data with that query. This can be done via the [with()](http://devdocs.io/laravel/api/4.2/illuminate/database/eloquent/model#method_with) method and is referred to as **eager loading**:

```php
# Eager load the author with the book
$books = \App\Book::with('author')->get();

foreach($books as $book) {
    echo $book->author->first_name.' '.$book->author->last_name.' wrote '.$book->title.'<br>';
}

dump($books->toArray());

```




## Tips
If you receive an error like this:

```txt
General error: 1005 Can't create table 'database-name.#sql-13993_88'
(errno: 150) (SQL: alter table `books` add constraint books_author_id_foreign foreign key
(`author_id`) references `authors` (`id`))`                                                                      
```

There may be two possible causes:

__Cause 1)__ When building relationships, it's important that you create tables in a logical order. For example: if the table `books` will have a FK connecting it to `authors`, then the `authors` table should exist before attempting to create that FK.

__Cause 2)__ When creating a column that will contain a FK relating it to another table, make sure that column is unsigned. Example:

```php
# Important! FK has to be unsigned because the PK it will reference is auto-incrementing
$table->integer('author_id')->unsigned();
```



## Reference
+ [Docs: Relationships](http://laravel.com/docs/eloquent#relationships)
+ [Docs: Relationships: One to Many](http://laravel.com/docs/eloquent-relationships#one-to-many)
+ [Docs: Migrations: Foreign Key Constraints](http://laravel.com/docs/migrations#foreign-key-constraints)
