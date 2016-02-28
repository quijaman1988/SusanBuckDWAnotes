See also: <https://laravel.com/docs/controllers>

---


## What are Controllers?
+ Controller classes make up the **C** of the **MVC** structure; they act as the glue between the request, the Model and the resulting View (or data).

+ For tiny applications it's convenient to throw all your logic into closures in your routes. As things grow, however, this becomes messy and hard to test.

+ Controllers offer a way to organize your routing logic.


## Structure
Example:
```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;

class BookController extends Controller {

	/**
	* Responds to requests to GET /books
	*/
	public function getIndex() {
        return 'List all the books';
	}

    /**
     * Responds to requests to GET /books/show/{id}
     */
    public function getShow($id) {
        return 'Show book: '.$id;
    }

    /**
     * Responds to requests to GET /books/create
     */
    public function getCreate() {
        return 'Form to create a new book';
    }

    /**
     * Responds to requests to POST /books/create
     */
    public function postCreate() {
        return 'Process adding new book';
    }
}
```

+ Put Controller class files `/app/Http/Controllers/`. This directory is psr-4 loaded in `composer.json`, so anything you put here will be readily available using the appropriate namespace.

+ You can technically call a controller class file anything you want, but it's a convention to suffix it with `Controller` (ex: `BookController`).

+ Your controller class should extend Laravel's `Controller`, which also exists in `/app/Http/Controllers/`. This is where you can put common logic shared by all your controllers.

+ Within your controller class you'll have public methods which represent the **actions** of your Controller.

+ One naming convention for your action methods is to prefix with the HTTP verb (`get,` `post` or `any`) . For example: `getSignup()` and `postSignup()`.

+ Each action method is defined in camelCase, which is required.

+ For actions with multiple words, dash-syntax on the URL will be translated to camelCase in the action. For example, a URL like `/generate-new-password` would translate to a method `getGenerateNewPassword`.




## Resource / RESTful Controllers
**REpresentational State Transfer (REST)** is a software design pattern commonly used by web applications.

There are lots of fancy explanations for REST, but really it comes down to this:

Every application has certain *things* that you'll want to perform CRUD operations on. For example, in Foobooks, one of our *things* is a Tag. Within our application we'll want to *Create tags*, *Update tags*, *Read tags* and *Delete tags*.

REST is a **convention for formatting the URLs** that will perform these operations.

### REST Expectations:

+ Your application will have a standard set of URLs to perform basic CRUD operations on a **Resource** (i.e. a *thing*).
+ A Resource is some entity of your application, for example a User or a Tag.
+ Every Resource has a unique ID.


### REST Example
For an example, let's build a RESTful interface for a Tag resource in the Foobooks application.

To start, you can have Artisan generate your Tag controller for you:

```bash
php artisan make:controller TagController --resource
```

If you open the resulting `TagController.php` you should see the following 7 methods:

+ `index()`
+ `create()`
+ `store()`
+ `show($id)`
+ `edit($id)`
+ `update($id)`
+ `destroy($id)`

Every RESTful controller should have these 7 methods to manage your basic CRUD operations.

According to REST conventions, your Routes would be defined like this:

```php
Route::get('/tag', 'TagController@index');
Route::get('/tag/create', 'TagController@create');
Route::post('/tag', 'TagController@store');
Route::get('/tag/{tag_id}', 'TagController@show');
Route::get('/tag/{tag_id}/edit', 'TagController@edit');
Route::put('/tag/{tag_id}', 'TagController@update');
Route::delete('/tag/{tag_id}', 'TagController@destroy');
```

[You can see a table of these routes in the Laravel docs](http://laravel.com/docs/controllers#restful-resource-controllers)

Note how some of the URI's are the same; this is okay because they use different HTTP requests.

You could explicitly define all of your REST routes like above, *or* you can do it all in one shot with Route's `resource()` method:

```php
Route::resource('tag', 'TagController');
```


### HTTP methods spoofing for methods other than GET and POST
HTML Forms only support GET and POST methods by default, but Laravel works around this by &ldquo;spoofing&rdquo; the `PUT` and `DELETE` methods. This is done by injecting a hidden field `_method` with your forms.

For example, this Form with the method `put`....

```php
{{ Form::model($tag, ['method' => 'put', 'action' => ['TagController@update', $tag->id]]) }}

	{{ Form::text('name') }}

	{{ Form::submit('Update') }}

{{ Form::close() }}
```

Will include this hidden field...

```html
<input name="_method" type="hidden" value="PUT">
```

### Destroy
Accessing the destroy action is a little funny, because in most cases you just want to link to it. You can't force a link to process via a method other than HTTP's GET.

```html
<!-- This will only ever use the GET method; how do we get it to use the DELETE method? -->
<a href='/tag/{{ $tag->id }}'>Delete</a>
```

To get around this, you'll have to create a mini-form with the DELETE method. Inside the form, you can embed a link that will submit the form (shown here with inline JS for simplicty):

```php
{{ Form::open(['method' => 'DELETE', 'action' => ['TagController@destroy', $tag->id]]) }}
	<a href='javascript:void(0)' onClick='parentNode.submit();return false;'>Delete</a>
{{ Form::close() }}
```
