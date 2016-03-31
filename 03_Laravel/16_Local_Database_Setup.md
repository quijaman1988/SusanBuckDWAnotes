## Starting assumption
+ You've already created a database, per the instructions in the Databases Primer notes. Our example database name is `foobooks`.
+ MAMP or XAMPP is open and your MySQL database server is running.

<img src='http://making-the-internet.s3.amazonaws.com/mysql-running-in-xampp-and-mamp@2x.png' style='max-width:921px; width:100%' alt='MySQL running in XAMPP and MAMP'>






## Laravel Database Configuration
To connect to your local MySQL server and database, Laravel needs your MySQL host name, username, and password as well as the name of the database you want to connect to.

These configurations are set in `/config/database.php`, so open that file.

For our examples we'll be using the MySQL database that comes with MAMP so find `mysql` in the `connections` array.

```php
'mysql' => [
    'driver'    => 'mysql',
    'host'      => env('DB_HOST', 'localhost'),
    'database'  => env('DB_DATABASE', 'forge'),
    'username'  => env('DB_USERNAME', 'forge'),
    'password'  => env('DB_PASSWORD', ''),
    'charset'   => 'utf8',
    'collation' => 'utf8_unicode_ci',
    'prefix'    => '',
    'strict'    => false,
],
```

Now how the `env()` method is used, making it easy to set separate database configurations for your different environments. This is important, because you'll be using two different databases&mdash; one locally and one live.

Open your local `.env` files and you should see the 4 `DB_` constants are set with some defaults:

```
DB_HOST=localhost
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret
```

You should update these values.

The default configs for MySQL in MAMP are:
+ Host: `localhost`
+ User: `root`
+ Password: `root`

(FYI These above values are found on the MAMP start page at <http://localhost/MAMP>)

The default configs for MySQL in XAMPP are:
+ Host: `localhost`
+ User: `root`
+ Password: (blank, no password)

(FYI The above values are found in `C:/xampp/passwords.txt`)

Here's an example configuration for XAMPP:

```
DB_HOST=localhost
DB_DATABASE=foobooks
DB_USERNAME=root
DB_PASSWORD=
```




## Test your connection
Here's a route you can put in your `routes.php` file to test that the database connection is working:

```php
Route::get('/debug', function() {

	echo '<pre>';

	echo '<h1>Environment</h1>';
	echo App::environment().'</h1>';

	echo '<h1>Debugging?</h1>';
	if(config('app.debug')) echo "Yes"; else echo "No";

	echo '<h1>Database Config</h1>';
    /*
    The following line will output your MySQL credentials.
    Uncomment it only if you're having a hard time connecting to the database and you
    need to confirm your credentials.
    When you're done debugging, comment it back out so you don't accidentally leave it
    running on your live server, making your credentials public.
    */
	//print_r(config('database.connections.mysql'));

	echo '<h1>Test Database Connection</h1>';
	try {
		$results = DB::select('SHOW DATABASES;');
		echo '<strong style="background-color:green; padding:5px;">Connection confirmed</strong>';
		echo "<br><br>Your Databases:<br><br>";
		print_r($results);
	}
	catch (Exception $e) {
		echo '<strong style="background-color:crimson; padding:5px;">Caught exception: ', $e->getMessage(), "</strong>\n";
	}

	echo '</pre>';

});
```

When you visit the resulting route (`http://localhost/debug`) you should see some debugging information, and if everything went well a message saying `Connection confirmed` followed by a list of the databases on your server.

If you see a red `Caught exception:` message, you have some troubleshooting to do.

<small>
Note: The above route is echo'ing a lot of content to the page directly from your routes file. This is only okay because it's a temporary route for debugging purposes.
</small>




## Conclusions
With your database setup and your connection confirmed, you're ready to move on to **Migrations** in order to build your tables.
