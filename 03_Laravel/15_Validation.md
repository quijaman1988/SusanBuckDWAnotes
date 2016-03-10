## Validation

Validation is an essential part of any application.

For an example of why, let's consider some of the critical ways a user sign up form could go wrong if validation is not applied:

+ Blank password
+ Blank email
+ Invalid email (ex: `sam@gmailcom`)
+ Duplicate email

Beyond critical issues, there may be added criteria you want to validate against such as a minimum length for a password.

With Laravel applications, the validation process is built in with plenty of room for customization.




## Demo setup
As a demo, we'll implement validation for the process of adding a new book in Foobooks.

At this point, our form for adding a new book looks like this:

```html
<form method='POST' action='/books/create'>
    {{ csrf_field() }}
    <input type='text' name='title'>
    <input type='submit' value='Submit'>
</form>
```

That form uses this route:

```php
Route::post('/books/create', 'BookController@postCreate');
```

Which uses this method in `BookController.php`:

```php
/**
 * Responds to requests to POST /books/create
 */
public function postCreate(Request $request) {
    $title = $request->input('title');
    return 'Process adding new book: '.$title;
}
```



## Validating the request
Below is a basic validation example where we validate that the incoming `title` for a new book is a) filled out (i.e. required) and b) at least three characters long:

```php
/**
 * Responds to requests to POST /books/create
 */
public function postCreate(Request $request) {

    // Validate the request data
    $this->validate($request, [
        'title' => 'required|min:3',
    ]);

    // If the code makes it here, you can assume the validation passed
    $title = $request->input('title');

    // Code would go here to add the book to the database

    // Then you'd give the user some sort of confirmation:
    return 'Process adding new book: '.$title;
}
```

Some notes about the above:

1. When using validation, be sure the `Request $request` object is passed to the method. The validator is run on this object, so it's necessary.
2. `$this->validate` is automatically available to your controller methods. You pass it the request object and an array of fields and validation rules.

At this point you can test out your validation:

1. Submit the form adding a title greater than 3 characters. You should see the message `Process adding new book: [book name]`.
2. Submit the form *without* adding a title, or adding a title that's less than 3 characters long. It won't show you your confirmation message, but will instead just take your straight back to the form.

At this point validation is working in that it's not finishing the `postCreate` method if the data is invalid. However, it's not very useful to the user since there's no indication that there was a problem. Your next step is to access and display some error feedback...




## Displaying errors
If validation fails, Laravel automatically redirects the user back to their previous location. In this process, Laravel also makes error messages available to your view in a variable called `$errors` which is an instance of `Illuminate\Support\MessageBag`.

Given this, you can display errors in your view like so:

```html
@if(count($errors) > 0)
    <ul>
        @foreach ($errors->all() as $error)
            <li>{{ $error }}</li>
        @endforeach
    </ul>
@endif

<form method='POST' action='/books/create'>
    <input type='hidden' name='_token' value='{{ csrf_token() }}'>
    <input type='text' name='title'>
    <input type='submit' value='Submit'>
</form>
```

In the above example, we're using the `all()` method on `$errors`, one of many methods available to us when dealing with the error MessageBag.

Alternatively, if we wanted *just* the errors related to the title field (assuming we had more than just the title field present), you could say:

```html
@if($errors->get('title'))
    <ul>
        @foreach($errors->get('title') as $error)
            <li>{{ $error }}</li>
        @endforeach
    </ul>
@endif
```

Read the section on [Working with Error Messages](http://laravel.com/docs/5.1/validation#working-with-error-messages) to see more about the methods available to you in the `$error` MessageBag.




## Available validation rules
In the above example, we only saw two rules: `required` and `min`. There are currently 42 total validation rules provided for you from Laravel - you can see and read all about them here: [Docs: Validation: Available Validation Rules](http://laravel.com/docs/5.1/validation#available-validation-rules).




## Read more
+ [Docs: Validation](http://laravel.com/docs/validation)
