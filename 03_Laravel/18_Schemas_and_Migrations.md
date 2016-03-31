With your application's database created and your connection to that database confirmed, it's time to build your tables.

Rather than building tables with raw SQL queries, or using a GUI like phpMyAdmin, we'll use **Laravel Migrations**.

__Migrations are PHP scripts that describe alterations to your database&mdash; whether it be adding tables, dropping tables, adding new columns to tables, etc.__

The benefits of managing your database with Migrations include...

+ The scripts can be kept in Version Control so you have a history of changes over time.
+ When you deploy your application to a new/different server, you can easily run your migrations to create a  copy of your database for that server.
+ Similarly, new developers can quickly build copies of your database on their local servers.

**Before proceeding, if you created a `books` table when reading the *Databases Primer* notes, delete that table now.**

<img src='http://making-the-internet.s3.amazonaws.com/laravel-how-to-delete-a-table-in-phpmyadmin@2x.png' style='max-width:1000px; width:100%' alt=''>


## Generate a new migration file
Laravel's command line utility, Artisan, has several commands to help with Migrations.

First, we'll prompt Artisan to generate a new migration file.

```bash
$ php artisan make:migration create_books_table
```

The migration name (`create_books_table`) should describe what you're doing. In this example we're creating a table called `books`.

After you run the above command, navigate to `/database/migrations/` where you should see a new file that's named something like `2015_10_28_050305_create_books_table.php`.

In `/database/migrations/` you'll also see 2 pre-existing migrations that come with Laravel by default: `create_users_table` and `create_password_resets_table`. These tables support Laravel's user authentication features, which will be discussed more at a later date.

Note that each migration filename is prefixed with a unique timestamp&mdash; this ensures migrations are run in the same order in which they were created.




## Write your first migration
Open up your the books migration file, and you should see something like this:

```php
<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateBooksTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        //
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        //
    }
}
```

Every generated migration has this boilerplate code with a `up()` and `down()` method.

In the `up()` method you'll describe some change to the database (adding a new table, adding fields to a table, etc.).

in the `down()` method you'll reverse any changes made in the `up()` method.




## Design the books table
Before writing any code in the `up()` method, we have to decide what fields the table will need, and what MySQL data type each field in the table should be.

To do this, we'll refer to the plan we came up with in the Database Primer notes:

<img src='http://making-the-internet.s3.amazonaws.com/laravel-books-table-design@2x.png' style='max-width:480px; width:100%' alt='books table design'>

To understand the MySQL data types chosen each field, [refer to this table](https://github.com/susanBuck/dwa15-fall2015-notes/blob/master/03_Laravel/15_Common_MySQL_Data_Types.md).

For all our tables/fields, we'll follow these suggested naming conventions:

+ Table names are plural.
+ Tables and field names are written in `snake_case`.

With the `books` table design in mind, it's time to write the Schema building code.




## up() - Build the table
Here's an example of a `up()` method to create the `books` table:

```php
public function up() {

	Schema::create('books', function (Blueprint $table) {

		# Increments method will make a Primary, Auto-Incrementing field.
		# Most tables start off this way
		$table->increments('id');

		# This generates two columns: `created_at` and `updated_at` to
		# keep track of changes to a row
		$table->timestamps();

		# The rest of the fields...
		$table->string('title');
		$table->string('author');
		$table->integer('published');
		$table->string('cover');
		$table->string('purchase_link');

		# FYI: We're skipping the 'tags' field for now; more on that later.

	});
}
```

Note how the Laravel `Schema` component is used to create the `books` table.

We've commented the `Schema` methods we're using, but be sure to read the [Schema Documentation](http://laravel.com/docs/migrations#creating-tables) for full details.




## down() - Drop the table
The &ldquo;undo&rdquo; action for creating a new table is really simple&mdash; just drop the table:

```php
public function down()
{
	Schema::drop('books');
}
```




## Run your migrations
When you've completed writing the code for your migration, you'll use Artisan to run it:

```bash
$ php artisan migrate
```

Example output:
```bash
Migration table created successfully.
Migrated: 2014_10_12_000000_create_users_table
Migrated: 2014_10_12_100000_create_password_resets_table
Migrated: 2015_10_28_050305_create_books_table
```

FYI: The first time you run `php artisan migrate` it will create a `migrations` table which will be used to keep track of what migrations you've run.

That should do the trick. Examine your new `books` table in phpMyAdmin to make sure it matches the design you were aiming for.

Moving forward, you should not make *any* structural changes to your database tables via phpMyAdmin. phpMyAdmin should now only be used to view tables and data. Any structural changes to tables need to be made with migrations. If you make changes via phpMyAdmin, your database/tables will be out of sync with your migrations.




## Altering tables
Any alterations to an existing table should be done in a new migration.

For example, let's imagine you wanted to add a `page_count` field to the books table. Start with a new migration:

```bash
$ php artisan make:migration add_page_count_field_to_books_table
```

Edit the resulting migration:

Up...

```php
public function up()
{
        Schema::table('books', function (Blueprint $table) {
            $table->integer('page_count');
        });
    }
```

Down...

```php
public function down()
{
	Schema::table('books', function (Blueprint $table) {
		$table->dropColumn('page_count');
	});
}
```

Run it:

```bash
$ php artisan migrate
```

Note how Artisan only runs this latest migration.
Any time you call `artisan migrate` it will only run migrations that have not already been run.




## Refresh
The artisan command to reset and re-run all migrations looks like this:

```bash
$ php artisan migrate:refresh
```

When you run this command it will go through your migrations in reverse, first executing all your `down()` methods.

Then, it will run through your migrations forward, executing all your `up()` methods.




## Starting over/Your first migrations
Getting your migrations right the first time can be challenging, especially if you're starting a new project with several new tables.

In many cases, you'll forget fields that you needed to add. Yes, you can create new migrations to add new fields, but starting off, it'd be nice to have single migrations for each new table, without a bunch of new migrations for each field you forget or mistake you need to fix.

There are several Artisan migrate commands that seem like they might help in this situation such as `migrate:refresh` and `migrate:reset` (ref: [Rolling Back Migrations](http://laravel.com/docs/5.1/migrations#rolling-back-migrations)).

These commands are useful but they assume your `down()` functions are written perfectly, and that you haven't had any migrations abort part-way through because of some issue. These assumptions aren't always correct, especially when you're first learning about migrations and building a lot of tables.

Given this, if you ever want to do an __absolute &ldquo;start over&rdquo;__ you can manually delete *all* your tables (including the `migrations` table) using phpMyAdmin. This will ensure you can run your migration with an absolute fresh start.

At this point, this work-around is acceptable because you're the only one running your migrations since the project hasn't been shared with anyone. It's only once your project has been shared with teammates/other servers that you obviously can't just start over, and you'll want to make sure even granular changes are done a migration at a time.

__Bonus tip:__

Here's a route you may want to implement when practicing/learning. It will quickly drop and re-create your foobooks database giving you an absolute fresh start. The same thing can be accomplished via phpMyAdmin, but this will get the job done with less clicks.

```php
if(App::environment('local')) {

    Route::get('/drop', function() {

        DB::statement('DROP database foobooks');
        DB::statement('CREATE database foobooks');

        return 'Dropped foobooks; created foobooks.';
    });

};
```

The above route has been wrapped in an if statement so it will only run in your local environment (to avoid accidentally deleting your production database).

Be very cautious with this route as it's very powerful. Only use it in prototypes, not in real world applications.
