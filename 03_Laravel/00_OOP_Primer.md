Laravel is built with Object Oriented PHP, so understanding the fundamentals of Object Oriented Programming (OOP) will help you better understand Laravel.

In the following OOP primer, I'll cover the essentials.

For a more in-depth look at OOP in PHP, refer to the following resources:

* [tutsplus.com: OOP PHP for Beginners](http://net.tutsplus.com/tutorials/php/object-oriented-php-for-beginners/)
* Skim or Bookmark: [PHP.net OOP](http://www.php.net/manual/en/language.oop5.php)


---

Object Oriented Programming (OOP) helps create clean, maintainable, reusable code.

OOP provides more organization and structure than *procedural programming* (*do this, then this, then this*).


## Classes
OOP Starts with **Classes**.

A Class defines an entity of your project, for example a User.

Classes are composed of two things: **properties** and **methods**.

Example: *A User has characteristics (properties) and it can do these things (methods)*.

Outside the context of OOP, *properties* are essentially *variables*, and *methods* are essentially *functions*.





## Objects
From Classes comes **Objects**.

If a Class is a cookie-cutter, the Object is the cookie.

Syntax for creating a class:

```php
class User {

	// Properties

	// Methods

}
```

Syntax for instantiating (creating) a new Object from a Class:

```php
$user = new User();
```


## Properties

```php
class User {

	// Properties
	public $name;
	public $email;
	public $logged_in = false;

	// Methods

}
```

+ Prefix with the keyword `public`, `private`, or `protected`
	+ **public** : property is accessible inside and outside the object
	+ **private** : property is only accessible inside the object
	+ **protected** : property is accessible inside the object and any objects which inherit this object
+ In PHP, declaring properties in your Class is optional - but it's good practice.
+ You can assign simple defaults to properties (ex: `15`, `false`, `foobar`), but you can not assign expressions (ex: `15 - 5`, `$foo * 2`).




## Methods

A method is just a function that belongs to a class/object.

Most Classes will start off with getter/setter methods that allow access to the Class properties.

```php
class User {

	// Properties
	public $name;
	public $email;
	public $logged_in = false;

	// Methods
	public function getName() {
		return $this->name;
	}
	public function setName($new_name) {
		$this->name = $new_name;
	}

}
```

+ Using getters/setters demonstrates **encapsulation**; they make it so outside code can't interact with your object's data without that object having full control.
+ `$this` is a built-in variable that is a reference to the current object's methods and properties.
+ `public` is an example of an **access modifier** which controls where an object's method can be accessed from.
+ Access modifier options:
	 + **public** : method is accessible inside and outside the object
	 + **private** : method is only accessible inside the object
	 + **protected** : method is accessible inside the object and any objects which inherit this object
	 + **static** : no object instantiation is needed; can be called directly on the class. This modifier can be added in addition to the `public`, `private` or `protected` modifiers. For example, you can say: `public static foobar() {}` to declare that a method is both `static` *and* `public`.


## Magic Methods

Classes can contain [PHP magic methods](http://php.net/manual/en/language.oop5.magic.php), which are predefined methods that have built-in functionality.

For example, your class can have a `__construct()` magic method. What makes this method &ldquo;magical&rdquo; is that it will automatially be called whenever you instantiate a new object of your class.

```php
class User {

	// Properties
	public $name;
	public $email;
	public $logged_in = false;


	// Methods

	function __construct() {
		echo 'You just created a new user!';
	}

	public function getName() {
		return $this->name;
	}
	public function setName($new_name) {
		$this->name = $new_name;
	}

}
```

Usage example:

```php
# This would output 'You just created a new user!';
$user = new User();
```

Given this, if you have statements that you want executed *everytime* a class is instantiated, put those statements in the construct method.




## Statically using classes
Generally, when using your Classes you first instantiate an Object from that class.

For example, if you had a Class called Email, you'd do something like this:

```php
$email = new Email();
```

And then maybe you'd write something like this (assuming your Email Class had a `send()` method):

```php
$email->send('jamal@gmail.com','Hello!');
```

Typically, instantiating Objects from Classes like this is the way to go.

However, you can also **statically** invoke a method from a Class, which skips the step of invoking an Object of that Class.

This is done using the class name, followed by `::`, then the method name. For example:

```php
Email::send('jamal@gmail.com','Hello!');
```

The downside of statically using Classes, is you don't have the benefit of storing properties with the object, which is one of the great advantages of OOP.

However, there are some instances where you might have a Class with some miscellaneous utility methods, that don't necessarily have related data. In that case, statically using that class is acceptable.
