Reports and Graphs
==================

[feature/reports-graphs](https://github.com/KLVTZ/Laracasts/tree/feature/reports-graphs)

Reports and Graphs are an essential aspect in representing data within your
site. Visually expressing data can provide a greater impact than just
meaningless numbers.

We will be taking advantage of [chart.js](http://chartjs.org). A free, and easy
to use library which uses the famous HTML5 canvas. To begin we pop in our
resource within our public folder as a JavaScript resource:
`public/js/vendor/chart.min.js`. We then begin by creating a simple controller
that will handle our daily report:

```php
Route::get('/', ['as' => 'home', 'uses' =>  function() { return 'Hello,
	Justin'; }]);

Route::get('admin/reports/daily', ['as' => 'admin.reports.daily', 'uses' =>
'ReportsController@daily'])->before('admin');
```

Our route will provide administrator like access. It will simple use an admin
filter to check that our request includes a query for our password. That is, a
simple get request is provided.

It's interesting to see that the Input facade can grab both POST data and GET
data.

```php
Route::filter('admin', function()
{
	if(Input::get('pwd') !== 'qfor')
		return Redirect::to('/');
});
```

In addition to these files, we also created a simple: `DailyReportsSeeder` and a
simple migration table.

Within our controller, we send in two array object collections. One with our
totals for each day and their respected dates. Notice we do a query for all
results and then send in a list, or an array, of the actual values. Not many
know about collection's lists method. We could send in both columns and have
date represented as an index and the value of each index be the actual total for
that day. But, in this example, we send in two lists. A list of dates and a
list of totals earned each day.

```php
<?php

class ReportsController extends \BaseController {

	public function daily()
	{
		$daily = DailyReports::all();

		return View::make('admin.reports.daily')
			->with('dates',	$daily->lists('date'))
			->with('totals', $daily->lists('amount'));
	}
}
```

Within our daily reports view, we create an empty canvas for our script to
inject a diagram into. We include a section for both the content as well as a
section for all our JavaScript. We yield the content in the master page, but we
only yield our JavaScript within our footer.

Chart.js
--------
We created an anonymous, self-invoked, function. This function grabs the canvas
tag and provides method and properties for drawing inside the canvas tag. We
then create a chart object with our data. We provide labels and datasets. The
labels themselves grab `json_encode` data as well as the data itself. Notice we
just use the provided arrays we pass through in our controller. We provide some
other information, which includes color information on how our graph will be
drawn. We finally create the new Chart object, instantiating it with our
context. We call the `Line` method with the option of chart and for it not to
curve. Alternatively, we could use `Bar` as a method call. Overall, Chart.js
offers a ridiculous amount of options for us to use when representing our data:

```php
@extends('layouts.master')

@section('content')
	<h1>Daily Reports</h1>
	<canvas id="daily-reports" width="700" height="300"></canvas>

	<h1>Drill Down</h1>
	@foreach ($totals as $index => $dailyIncome)
		<li><strong>{{ $dates[$index] }}:</strong> ${{$dailyIncome }}</li>
	@endforeach
@stop

@section('js-footer')
	<script src='/js/vendor/chart.min.js'></script>

	<script>
		(function() {
			var ctx = document.getElementById('daily-reports').getContext('2d');
			var chart = {
				'labels': {{ json_encode($dates) }},
				'datasets': [{
					data: {{ json_encode($totals) }},
					fillColor: "#f8b1aa",
					strokeColor: "#bb574e",
					pointColor: "#bb574e"
				}]
			};
			new Chart(ctx).Line(chart, {bezierCurve: false});
		 })();
	</script>
@stop
```
