Presenters
==========

[feature/presenters](https://github.com/KLVTZ/Laracasts/tree/feature/presenters)

Many times, when we are working with views, we want to control the way our
data is presented. We understand that it is not the responsibility of the view
to contain such logic. As is the same for our models. In reality, view specific
logic should be separated to it's own bounded context. As a result, presenters
are an option for isolating custom data accessories.

We begin with the creation of a Presenters folder that will be dedicated to
specific presenters. We take advantage of [PSR-0 Standards](http://www.php-fig.org/psr/psr-0/).

Be sure to load up your namespace in `composer.json`
```js
"psr-0": {
	"Acme": "app/"
}
```

Within our `app/routes.php` file, we provide a single closure for an index
request. Within that lambda, we grab a collection of users. We then return a
view that provides a user object. But we are decorating that users object with a
new instance of a Presenter Collection. We pass through two arguments: a User
Presenter, and an Eloquent users collection.

```php
<?php

Route::get('/', function()
{
	$users = User::all();

	return View::make('home')->with('users', new
		Acme\Presenters\PresenterCollection('Acme\Presenters\UserPresenter', $users));
});
```

Let's first discuss what we are passing through. Our first argument is a User
Presenter. A Presenter is dedicated to presenting an object's information in a
custom manner. The view is not concerned as to how that is done. It is the
responsibility of the presenter to create the presentation that the view will
render.

Every presenter will extend an abstract Presenter class. This class will
maintain a protected instance of a resource --in this instance, an actual user
object. We will also provide a magic method of `__get`. This magic method will
take the method name and see if it exists within the current object instance.
If it does, it will directly return the value that is called from that method.
In our example, this condition will prove true for methods that we add to our
specific presenter --such as the User Presenter. Else, that custom user
presenter does not contain that method call and therefore must belong directly
the resource that we pass through --the user object.

```php
<?php namespace Acme\Presenters;

abstract class Presenter
{
	protected $resource;

	public function __construct($resource)
	{
		$this->resource = $resource;
	}

	public function __get($name)
	{
		if (method_exists($this, $name))
		{
			return $this->{$name}();
		}

		return  $this->resource->{$name};
	}
}
```

Within our `UserPresenter` class, we extend the abstract `Presenter` class. We
provide two custom methods that we wish to provide to our decorated collection.
A gravatar method that returns a path as well as a custom, formatted, created_at
data stamp.

```php
<?php namespace Acme\Presenters;

class UserPresenter extends Presenter
{
	public function gravatar()
	{
		return 'gravatar path';	
	}

	public function created_at()
	{
		return $this->resource->created_at->format('Y-m-d');
	}
}
```

This is what is passed directly to the `PresenterCollection`. Moving onto it,
this is where the majority of our features can be found. We begin by passing
through the presenter (that `User Presenter` class) and that of the user object
collection. Within each iteration of the collection, we have an index that
points to a user object. Our goal is to wrap that object with its presenter and
thus provide the additional custom methods in place of a user object.

To do this, we update the key value with a new instance of the presenter that
contains the original user object. By doing so, each index is updated with a
presenter that decorates a user object. We update our own reference of this
collection to a protected property.

Because we want the collection to be countable, accessible as an array, and
aggregated, we take advantage of PHP interfaces that provide such functionality.

For a countable object, we return a count of the collection. For an iterative
object, we provide the collection in which we want to iterate over. Lastly, to
make the object accessible as an array, we must implement four methods. The
`offsetExists` method checks to see if our request offset exists and returns a
boolean. Our `offsetGet` returns the key value, if it does indeed exist. The
`offsetSet` allows us to set an offset value. And lastly, our `offsetUnset` will
unset an object collection. 

```php
<?php namespace Acme\Presenters;

use Illuminate\Database\Eloquent\Collection;

class PresenterCollection implements \Countable, \ArrayAccess, \IteratorAggregate
{
	protected $collection;

	public function __construct($presenter, Collection $collection)
	{
		foreach ($collection as $key => $resource) {
			$collection->put($key, new $presenter($resource));
		}
		$this->collection = $collection;
	}

	public function count()
	{
		return count($this->collection);
	}

	public function getIterator()
	{
		return $this->collection;
	}

	public function offsetExists($offset)
	{
		return isset($this->collection[$offset]);
	}

	public function offsetGet($offset)
	{
		return isset($this->collection[$offset]) ? $this->collection[$offset] : null;
	}

	public function offsetSet($offset, $value)
	{
		$this->collection[$offset] = $value;	
	}

	public function offsetUnset($offset)
	{
		unset($this->collection[$offset]);
	}
}
```

The end result allows us to directly call the gravatar method from within a
collection of user object. Notice that it is not the responsibility of the view
nor the model to deal with the logic of how this information is rendered.
Instead, view specific logic is dedicated to a view.

```php
@foreach ($users as $user)
	{{ $user->gravatar }}
@endforeach
```

Now we have this contained area for presenting a user. Or let's say we have a
post model, we can have a post model that is dedicated to displaying those post. 
