Database seeding is the process of entering sample/testing data into your tables.

In Laravel this can be done via scripts called __seeders__.

Benefits:
+ Good for __development environments__, where you don't have real data to work with yet.
+ Good for __testing__, so that your testing scripts has disposable data to work with.
+ Good for __collaboration__; if another developer (or TA or instructor) clones your project they can quickly build their tables using your migrations, and fill those tables with sample data using your seeders.




## Create your first seeder
Let's start by creating a new seeder for the books table.

There's an artisan command `make:seeder` that can be used to generate a boilerplate seeder class:

```bash
php artisan make:seeder BooksTableSeeder
```

This will generate a new file in `/database/seeds/BooksTableSeeder.php` which includes one method, `run()`:

```php
<?php

use Illuminate\Database\Seeder;

class BooksTableSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        //
    }
}
```

This `run()` method is invoked when this seeder is called. Given that, the `run()` method is where you'll put the code that will actually enter seed data into the *books* table.

Here's some sample code showing how to add three books in the `run()` method:

```php
public function run()
{
    DB::table('books')->insert([
        'created_at' => Carbon\Carbon::now()->toDateTimeString(),
        'updated_at' => Carbon\Carbon::now()->toDateTimeString(),
        'title' => 'The Great Gatsby',
        'author' => 'F. Scott Fitzgerald',
        'published' => 1925,
        'cover' => 'http://img2.imagesbn.com/p/9780743273565_p0_v4_s114x166.JPG',
        'purchase_link' => 'http://www.barnesandnoble.com/w/the-great-gatsby-francis-scott-fitzgerald/1116668135?ean=9780743273565',
    ]);

    DB::table('books')->insert([
        'created_at' => Carbon\Carbon::now()->toDateTimeString(),
        'updated_at' => Carbon\Carbon::now()->toDateTimeString(),
        'title' => 'The Bell Jar',
        'author' => 'Sylvia Plath',
        'published' => 1963,
        'cover' => 'http://img1.imagesbn.com/p/9780061148514_p0_v2_s114x166.JPG',
        'purchase_link' => 'http://www.barnesandnoble.com/w/bell-jar-sylvia-plath/1100550703?ean=9780061148514',
    ]);

    DB::table('books')->insert([
        'created_at' => Carbon\Carbon::now()->toDateTimeString(),
        'updated_at' => Carbon\Carbon::now()->toDateTimeString(),
        'title' => 'I Know Why the Caged Bird Sings',
        'author' => 'Maya Angelou',
        'published' => 1969,
        'cover' => 'http://img1.imagesbn.com/p/9780345514400_p0_v1_s114x166.JPG',
        'purchase_link' => 'http://www.barnesandnoble.com/w/i-know-why-the-caged-bird-sings-maya-angelou/1100392955?ean=9780345514400',
    ]);
}
```

The above code is using Laravel's [QueryBuilder](http://laravel.com/docs/queries) to insert the books.

__Carbon Time__

For the `created_at` and `updated_at` fields, we used the `Carbon::now()->toDateTimeString()` method to generate a new timestamp based on the current time.

Carbon is a package that comes with Laravel and provides many date/time methods; you can read more about Carbon here: <https://github.com/briannesbitt/Carbon>.

Using `Carbon::now()` as we have in this example will give all of your books the same `created_at`/ `updated_at` values. If you need your sample data to have unique timestamp fields, you could use a package like [Faker](https://github.com/fzaninotto/Faker) to generate random time strings.









## Master seeder
Next, open `database/seeds/DatabaseSeeder.php` where you should see this existing code:

```php
<?php

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        // $this->call(UserTableSeeder::class);
    }
}
```

This `DatabaseSeeder` class is the master seed file that gets called when you run `php artisan db:seed`. It's job is to invoke your individual Seeder classes.

Note there's a sample invocation commented out:

```php
//$this->call(UserTableSeeder::class);
```

Let's uncomment that, and have it invoke the `BooksTableSeeder`:

```php
$this->call(BooksTableSeeder::class);
```

Finally, run `php artisan db:seed` to run the master seeder, which should invoke the `BooksTableSeeder`.

You should see a result like this:

```
$ php artisan db:seed
Seeded: BooksTableSeeder
```

Open phpMyAdmin to view your *books* table and confirm your seed data was added.



## Running seeds again

If you call `php artisan db:seed` again, it will rerun your seeds, creating duplicate rows, which may not be what you want.

To prevent duplicate rows, you could manually empty your database tables via phpMyAdmin, then run the seeders again.

__Or__, you could run `php artisan migrate:refresh` to rebuild all your tables, effectively emptying them.
Then you can run `php artisan db:seed` again.

__Or__, you can do the last two steps in one with the `migrate:refresh` command followed by the `--seed` flag:

```bash
$ php artisan migrate:refresh --seed
```




## Reminder: Seeding in P4
In addition to migrations, it's required that your P4 utilizes database seeding. This practice makes it quick for us to easily build your database tables and quickly fill them with sample data so that we can more efficiently help you troubleshoot.



## Read More
+ <http://laravel.com/docs/seeding>
