In the notes that come after this, we'll be discussing Routing, which is the first instance where we'll see Laravel use something called **facades**.

Facades are used throughout Laravel, so before we go any further we should cover what a facade is.

__A facade is a &ldquo;static proxy&rdquo; to a underlying class provided by Laravel.__

Here's an example using Laravel's `Hash` facade:

```php
Hash::make('secretpassword');
```

This syntax&mdash; facade name (`Hash`), followed by two double colon (`::`), plus some method (`make()`), will be used frequently throughout your code when working with Laravel.

If you have experience with Object Oriented Programming or read the OOP Primer, you know the double colon means you're **statically** using a class. That is, you're invoking a method from a class, without instantiating an object from that class.

For example, instead of instantiating an object from the class `Hash` like this:

```php
$hash = new Hash();
echo $hash->make('secretpsasword');
```

You're statically using that `Hash` class:

```php
echo Hash::make('secretpassword')
```

Given that knowledge, you may think the facades and double quotes used throughout Laravel is an excessive use of static method invoking, which is something you generally try to avoid.

However!

In this instance, it's not actually a static invocation. It's simply a static-*looking* proxy to an actual instance of that class that exists in the Laravel app.

Here's how the Laravel documentation explains facades:

>> *Facades provide a &ldquo;static&rdquo; interface to classes that are available in the application's IoC container. Laravel ships with many facades, and you have probably been using them without even knowing it! Laravel "facades" serve as "static proxies" to underlying classes in the IoC container, providing the benefit of a terse, expressive syntax while maintaining more testability and flexibility than traditional static methods.* -[src](http://laravel.com/docs/facades#facade-class-reference)


## Facades and their classes
**Understanding that the facades you'll use throughout Laravel are really just static proxies for instantiated classes is important, because you'll want to know what those classes are in order to look up how to use them.**

To help with this, here's a [list of facades and their underlying classes](http://laravel.com/docs/facades#facade-class-reference).

For example:
`Route` is a Facade for the `Router` class. Once you know that, it makes finding information in the API docs easier: [API: Router](http://devdocs.io/laravel/api/5.2/illuminate/routing/router).


## More info...

To learn more about the facade design pattern check out this article: [What Are Laravel 5.0 Facades?](http://code.tutsplus.com/tutorials/what-are-laravel-50-facades--cms-25347)


> &ldquo;*[...] the facade design pattern is a structural pattern that defines a simplified interface to a more complex subsystem. The pattern is based on creating a simple facade interface in front of the collection of required logic and methods. The facade itself maintains the dependencies.*&rdquo;
