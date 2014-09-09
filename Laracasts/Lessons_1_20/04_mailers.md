Mailers
=======

[feature/mailers](https://github.com/KLVTZ/Laracasts/tree/feature/mailers)

In this screen cast, we set out to create a mailer class that separates the
functionality of mailing items to a user and that of the settings. We will see
how to arrange our classes so that we can easily send emails to our users.

First, we need to adjust our `app/config/mail.php` file. That PHP file will
contain all the information you need to input for easy to use SMTP submission.
This will work by default when using the Mail object, however, we can do better.

I'm going to walk step-by-step through the route files, controllers, as well as
the mailers folder.

First, within our routes, we create a resource in which we register a RESTful
route to be available to our application. We override the store method for a
particular route. That is why we post it above the resource calling. So when the
user visits `http:// 127.0.0.1/users/create`, they will be directed by that
route.

```php
<?php


Route::get('users/store', 'UsersController@store');
Route::resource('users', 'UsersController');
```

Within our controller, we are calling the namespace `Acme\Mailers\UserMailer` as
`Mailer`. This means that we are technically using all functions within the
`User Mailer` file that we will be looking at later. However, this `User Mailer`
file inherits from the abstract class called `Mailer`. This means that we have
access directly to the file we are using and its inheritance. We use `Mailer`
because we already use it throughout the file. But technically, again, you're
calling the namespace of `User Mailer`. We then create a protected variable
within the `Users Controller` that will be used to reference the Mailer
namespace. That way, we can have a local instance every time the `Users
Controller` is rendered. We demonstrate this in the constructor.

```php
<?php

use Acme\Mailers\UserMailer as Mailer;

class UsersController extends BaseController {
	
	protected $mailer;

	public function __construct(Mailer $mailer)
	{
		$this->mailer = $mailer;
	}
```

Now drawing ourselves back to the store method, we first do a simple query tha
finds a user by their identifier.

*you can use PHP Artisan tinker to interact live on your application. You can
create a new user and save it's data to a db.*

We grab that data through the above query. We then call the reference of our
`User Mailer` class to call welcome and send it the user object.

```php
	public function store()
	{
		$user = User::find(1);
		$this->mailer->welcome($user);
	}
```

Within our `User Mailer` class, we extend the `Mailer` abstract class to receive
all its property functions. We call `User` so we can direct access to the user.
Because we send user data through, we want to make sure we are interacting with
the same object! That is why we call `User $user` --we are type-hinting. We then
create a few variables to send over --including a view. All this is called
within the send function.

By separating the welcome function and send to, we separate features from
functionality of sending an email. We provide an additional cancellation method
to demonstrate how we can categorize our methods to use our mailer abstract
class.

```php
<?php namespace Acme\Mailers;

use User;

class UserMailer extends Mailer {
	public function welcome(User $user)
	{
		$view = 'emails.welcome';
		$data = [];
		$subject = 'Welcome to Index!';

		return $this->sendTo($user, $subject, $view, $data);

	}

	public function cancellation()
	{
		$view = 'emails.cancel';
		$data = [];
		$subject = 'Sorry to see Index!';

		return $this->sendTo($user, $subject, $view, $data);
	}
}
```


Lastly, within our abstract class of `Mailer`, we are using the `Mail` namespace
and we are grabbing all the parameter arguments from within our function. We
can call mail send or queue. Either one will work, later on we will look at queue. 
This will make our process faster to send emails instead of waiting for an email
to submit. For now, we send in our view, data, and closure function. Notice, we
are sending in a message and then claiming to use the user and subject.
Remember, the closure is a callback function that will run on callback with the
use of user and subject. Very similar to JavaScript. We run the unction and then
run this closure function for th callback. Notice that we are not returning so
this is a closure for running on callback.

```php
<?php namespace Acme\Mailers;

use Mail;

abstract class Mailer {
	public function sendTo($user, $subject, $view, $data = [])
	{
		Mail::queue($view, $data, function($message) use($user, $subject)
	{
		$message->to($user->email)
			->subject($subject);
	});

	}
}
```
