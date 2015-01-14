Atom feeds decoded
==================

[feature/rss](https://github.com/KLVTZ/Laracasts/tree/feature/rss)

In this screen cast, we decoded Atom Feeds. We first grab our currently posted
data, parse through it, posting it to a blade template that the RSS feed then
displays. We send a response view so that we can make an XML or Atom feed.

First, we register a simple route to get a request for the feed of our website.
This will, in turn alolow us to use our controller:

```php
<?php

Route::get('feed', 'RssController@index');
```

Next, within our controller, create an array of data with a named index of
lessons. We assign this value the name spaced table of lesson and we order by
published date. We organize this in descending order by getting only 10 of the
most recent lessons. We return a resoinse view which will allow us to view a
page source of 200 --specifically expecting a render of XML data.

```php
<?php

class RssController extends BaseController {

	public function index()
	{
		$data['lessons'] = 'hello';//Laracasts\Lesson::orderBy('published_at', 'desc')->limit(10)->get();

		return Response::view('rss', $data, 200, [
			'Content-Type' => 'application/atom+xml; charset=UTF-8',
		]);
	}
}
```

Next, within our RSS view page, we create an XML styled sheet that provides us a
link to our website. We also provide a call to the name spaced carbon now for
atom string that will provide a dynamically added date for when something is
updated. We provide a normal for-each loop that will display information for
each feed provided. This includes: link to the lesson, a summary, and title. All
this information is provided through our lesson data to which, again, provided
from our array lessons index. Going through each index and output RSS
headings. 

```php
{{ '<?xml version "1.0" encoding="utf-8"?>' }}

<feed xmlns="http://www.w3.org/2005/Atom">
	<title>Laracasts</title>
	<subtitle>The best source of Laravel Training on the web.</subtitle>
	<link href='http://laravasts.com/feed' />
	<updated>{{ Carbon\Carbon::now()->toATOMString() }}</updated>
	<author>
		<name>Justin Page</name>
	</author>
	<id>tag:laracasts.com,{{ date('Y') }}:/feed</id>
	
	@foreach($lessons as $lesson)
		<entry>
			<title>{{ $lesson->title }}</title>
			{{ link_to_route('lessons.show', $lesson->slug) }}
			<id>...</id>
			<summary>{{ $lesson->excerpt }}</summary>
		</entry>
	@endforeach
</feed>
```
