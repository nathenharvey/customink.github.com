---
layout: post
title: "Profiling OpenStruct, Eager Loading, Method Missing, and Lazy Loading"
date: 2012-06-18 21:32
comments: true
published: true
author: seth-vargo
categories:
  - benchmarking
  - profiling
  - exploration
  - openstruct
  - eager loading
  - method_missing
  - lazy loading
  - optimization
---

I was recently working on a gem that involved marshaling data from a remote API. I really wanted the gem to behave like a native Ruby object, but they methods would vary depending on the response. Since the data was dynamic, it would have been counter-productive and not scalable to define each of the methods individually.

As such, I thought of a few different ways to get the result I wanted, but that just raised more questions, mainly **performance**. How would each of these methods perform? This was especially important given the large number of queries I would be making. In this blog post, I detail the exploration of these different methods and arrive at some pretty cool conclusions.


<!-- more -->
The Scenario
------------
In this scenario, we will be using the result of a [Github API call to my profile](https://api.github.com/users/sethvargo/repos). The hash (as follows) will be stored and referenced in the `@store` variable:

```ruby
{
  'has_wiki' => false,
  'has_issues' => false,
  'forks' => 0,
  'open_issues' => 0,
  'language' => 'Ruby',
  'description' => 'A distributed build system for the open source community.',
  'svn_url' => 'https://github.com/sethvargo/travis-ci',
  'pushed_at' => '2012-06-16T17:32:35Z',
  'full_name' => 'sethvargo/travis-ci',
  'git_url' => 'git://github.com/sethvargo/travis-ci.git',
  'created_at' => '2012-06-16T17:00:29Z',
  'url' => 'https://api.github.com/repos/sethvargo/travis-ci',
  'has_downloads' => true,
  'watchers' => 1,
  'size' => 188,
  'homepage' => 'http://travis-ci.org',
  'clone_url' => 'https://github.com/sethvargo/travis-ci.git',
  'ssh_url' => 'git@github.com:sethvargo/travis-ci.git',
  'html_url' => 'https://github.com/sethvargo/travis-ci',
  'updated_at' => '2012-06-16T17:32:35Z',
  'owner' => {
    'avatar_url' => 'https://secure.gravatar.com/avatar/87f282c6c2cdad13100dffe8c1daf77d?d=https://a248.e.akamai.net/assets.github.com%2Fimages%2Fgravatars%2Fgravatar-140.png',
    'login' => 'sethvargo',
    'gravatar_id' => '87f282c6c2cdad13100dffe8c1daf77d',
    'url' => 'https://api.github.com/users/sethvargo',
    'id' => 408570
  },
  'name' => 'travis-ci'
}
```

### The Metrics
Because of the nature of my curiosity, we will measure on three different metrics:

- creating the object
- initial request time
- average request time for requests (2-100)


### Pure Hash
The Pure Hash is our baseline. This is without any magic. We are just using the straight-forward bracket-notation for accessing a value in the hash:

```ruby
@store[key]
```

### OpenStruct
The [OpenStruct](http://ruby-doc.org/stdlib-1.9.3/libdoc/ostruct/rdoc/OpenStruct.html) is a Ruby 1.9 library that allows builds methods from a given hash.

`OpenStruct` certainly has advantages, being only two lines of code. It also returns `nil` instead of raising an exception when calling a "method" that doesn't exist.

```ruby
require 'ostruct'
class FiddleOpenStruct < OpenStruct; end
```

### Eager Loading
With Eager Loading, we parse the data on initial object creation and define methods for each of the objects in the hash. This is obviously expensive on the object creation, but should be faster for subsequent calls.

Eager Loading will raise an exception when calling a method that does not exist, although this could be overridden with `method_missing`.

```ruby
class FiddleEagerLoading
  def initialize(hash)
    hash.each do |key,value|
      define_singleton_method(key.to_s){ hash[key] }
    end
  end
end
```

### method_missing
Using `method_missing`, we "proxy" the requests to their associated keys in the hash store. The biggest problem with `method_missing` is performance, and that fact is very well-documented. Every call must go through the entire object stack before it hits `method_missing`, so we can except this method to not perform well.

With `method_missing`, we can either call `super` (raise an exception) or end with `nil` and any non-existent methods will return `nil`, just like `OpenStruct`.

```ruby
class FiddleMethodMissing
  def initialize(hash)
    @hash = hash
  end

  def method_missing(m, *args, &block)
    @hash[m.to_s].nil? ? super : @hash[m.to_s]
  end
end
```

### Lazy Loading
A hybrid of `Eager Loading` and `method_missing`, in Lazy Loading, we dynamically create methods as they are requested in `method_missing`. On an initial request, no methods exist, so we hit `method_missing`. However, `method_missing` defines a method. On a subsequent call, we don't execute the entire stack and call the newly created method (meta-programming for the win)!

```ruby
class FiddleLazyLoading
  def initialize(hash)
    @hash = hash
  end

  def method_missing(m, *args, &block)
    unless @hash[m.to_s].nil?
      value = @hash[m.to_s]
      define_singleton_method(m.to_s){ value }
      return value
    else
      super
    end
  end
end
```

The Test Suite
--------------
This test suite is very simplistic, coming in under 50 lines of code. You can see we are testing the three different metrics with appropriate `puts` statements to differentiate the output:

```ruby
@methods = @store.keys
@n = 10000

puts "\n\n"
puts 'Creating the Object'
Benchmark.bm(15) do |x|
  x.report('pure hash:')      { @n.times{ @store } }
  x.report('open struct:')    { @n.times{ FiddleOpenStruct.new(@store) } }
  x.report('eager loading:')  { @n.times{ FiddleEagerLoading.new(@store) } }
  x.report('method_missing:') { @n.times{ FiddleMethodMissing.new(@store) } }
  x.report('lazy loading:')   { @n.times{ FiddleLazyLoading.new(@store) } }
end

puts "\n\n"
puts 'First method call'
Benchmark.bm(15) do |x|
  x.report('pure hash:')      { @n.times{ @methods.each{|m| @store[m] } } }
  x.report('open struct:')    { @n.times{ @methods.each{|m| FiddleOpenStruct.new(@store).send(m.to_sym) } } }
  x.report('eager loading:')  { @n.times{ @methods.each{|m| FiddleEagerLoading.new(@store).send(m.to_sym) } } }
  x.report('method_missing:') { @n.times{ @methods.each{|m| FiddleMethodMissing.new(@store).send(m.to_sym) } } }
  x.report('lazy loading:')   { @n.times{ @methods.each{|m| FiddleLazyLoading.new(@store).send(m.to_sym) } } }
end

puts "\n\n"
puts "[2..#{@n}] times"
@fiddle_open_struct = FiddleOpenStruct.new(@store)
@fiddle_eager_loading = FiddleEagerLoading.new(@store)
@fiddle_method_missing = FiddleMethodMissing.new(@store)
@fiddle_lazy_loading = FiddleLazyLoading.new(@store)

@methods.each do |m|
  @store[m]
  @fiddle_open_struct.send(m.to_sym)
  @fiddle_eager_loading.send(m.to_sym)
  @fiddle_method_missing.send(m.to_sym)
  @fiddle_lazy_loading.send(m.to_sym)
end

Benchmark.bm(15) do |x|
  x.report('pure hash:')      { @n.times{ @methods.each{|m| @store[m] } } }
  x.report('open struct:')    { @n.times{ @methods.each{|m| @fiddle_open_struct.send(m.to_sym) } } }
  x.report('eager loading:')  { @n.times{ @methods.each{|m| @fiddle_eager_loading.send(m.to_sym) } } }
  x.report('method_missing:') { @n.times{ @methods.each{|m| @fiddle_method_missing.send(m.to_sym) } } }
  x.report('lazy loading:')   { @n.times{ @methods.each{|m| @fiddle_lazy_loading.send(m.to_sym) } } }
end
```

Results
-------
And the moment you have been waiting for - the results:

    Creating the Object
                          user     system      total        real
    pure hash:        0.000000   0.000000   0.000000 (  0.000515)
    open struct:      1.440000   0.000000   1.440000 (  1.440354)
    eager loading:    0.470000   0.000000   0.470000 (  0.468859)
    method_missing:   0.000000   0.000000   0.000000 (  0.003673)
    lazy loading:     0.000000   0.000000   0.000000 (  0.002815)


    First method call
                          user     system      total        real
    pure hash:        0.030000   0.000000   0.030000 (  0.021060)
    open struct:     32.720000   0.010000  32.730000 ( 32.740407)
    eager loading:   10.470000   0.030000  10.500000 ( 10.496159)
    method_missing:   0.270000   0.000000   0.270000 (  0.261547)
    lazy loading:     0.970000   0.000000   0.970000 (  0.976031)


    [2..10000] times
                          user     system      total        real
    pure hash:        0.020000   0.000000   0.020000 (  0.021658)
    open struct:      0.080000   0.000000   0.080000 (  0.077467)
    eager loading:    0.080000   0.000000   0.080000 (  0.081599)
    method_missing:   0.190000   0.000000   0.190000 (  0.191151)
    lazy loading:     0.080000   0.000000   0.080000 (  0.071562)

As expected, the pure hash (base) out-performed all other methods. In creating the initial object, the `OpenStruct` performed very poorly at over a second. Furthermore, as you might expect, eager loading took about half a second. The rest of the methods had negligible results. On the first method call, the `OpenStruct` took over 30 seconds, and eager loading took about 10 seconds. We also see that lazy loading began pulling ahead of `method_missing`. Finally, in subsequent requests (after initial object creation and first method call), `method_missing` performed the poorest.

Here are some observations to help you visualize the results:


### Creating Initial Object

                           0        10       100      1000     10000
    pure hash       0.000007  0.000003  0.000008  0.000052  0.000514
    open struct     0.000002  0.001660  0.014470  0.147943  1.507701
    eager loading   0.000002  0.000406  0.004800  0.046868  0.482254
    method_missing  0.000003  0.000007  0.000029  0.000269  0.003883
    lazy loading    0.000002  0.000007  0.000029  0.000535  0.002656

<div id="chart_creating_initial_object" style="width:100%; height:500px; min-width:600px;"></div>
<br />

Clearly the `OpenStruct` took a significant amount of time, measuring over 4x its closest competitor. Ignoring the `OpenStruct` outlier, eager loading, as expected, took longer than all other methods. This is because it generates all those methods on each object creation. Comparing the other results, we can conclude the following:

- **Don't use OpenStruct**
- **If you are creating a lot of objects, avoid Eager Loading (for now)**


### First Method Call

                           0        10       100      1000      10000
    pure hash       0.000008  0.000026  0.000236  0.002248  0.0210930
    open struct     0.000002  0.032629  0.328859  3.340419  33.446652
    eager loading   0.000003  0.011754  0.103952  1.072228  10.856142
    method_missing  0.000002  0.000329  0.003961  0.027261  0.2574720
    lazy loading    0.000002  0.000968  0.009337  0.100895  1.0136820

<div id="chart_first_method_call" style="width:100%; height:500px; min-width:600px;"></div>
<br />

Again we see that `OpenStruct` performs poorly, taking over 30 seconds to process 10,000 first method calls. `method_missing` surprisingly performed very well. Lazy loading doesn't have a chance to shine here because we are only calling the method once (thus essentially using `method_missing`). The extra time is from actually defining the method. With this data, we can conclude the following:

- **Don't use OpenStruct**
- **`method_missing` wins when only calling a few methods**


### Subsequent Method Calls

                           0        10       100      1000     10000    100000    1000000    10000000
    pure hash       0.000008  0.000027  0.000229  0.002292  0.021919  0.210670  2.1555960   21.669286
    open struct     0.000003  0.000090  0.000825  0.008313  0.078974  0.781603  7.9774830   76.648551
    eager loading   0.000002  0.000093  0.000878  0.008558  0.084123  0.839965  8.5130510   80.623932
    method_missing  0.000002  0.000217  0.001972  0.020136  0.186405  1.916927  19.288969  188.095461
    lazy loading    0.000002  0.000081  0.000754  0.007333  0.072564  0.737112  7.4007320   72.202056

<div id="subsequent_method_calls" style="width:100%; height:500px; min-width:600px;"></div>
<br />

In drawing conclusions from this data, it's more important to look at *how* the data is scaling, rather than the factors themselves. In the long run, lazy loading performed the best, but OpenStruct is a close second. It turns out OpenStruct isn't that bad when making a lot of queries. `method_missing` performed very poorly.


Final Conclusions
-----------------
Each of the methods we analyzed provide different benefits under different circumstances. It's impossible to say "use method x, because it's better". One of the reasons I chose to analyze multiple use cases is because it illustrates the fact that there is no "silver bullet" answer to this common problem.

If you're in `irb` and need to marshall a hash into an object, `OpenStruct` is the easiest and fastest route. Similarly, if you are creating an object that receives thousands of requests, you may want to implement lazy loading. There is not easy answer, but hopefully these statistics and benchmarks help in your next project.


Known Caveats
-------------
These methods are purely for example and fails many edge cases.

1. These methods require special cases when your object returns data that is actually `nil` or `null`. It could raise an exception instead of actually returning `nil` as expected.
2. Allowing methods to by dynamically called opens you up to a world of hurt - especially if your data's keys correspond to existing Ruby methods. As an example, when writing this blog post, my original hash had a key named `fork`. When I was actually sending `fork`, it was calling the Ruby `Kernel.fork` method, which really threw me for a loop.
3. I neglected many qualitative metrics, such as easy of use, implementation time, and readability, when conducting this study. Those metrics, by definition, are extremely difficult to measure.

<script type="text/javascript">
  google.load('visualization', '1', { packages:['corechart'] });
  google.setOnLoadCallback(drawCharts);

  var options = {
    hAxis: {
      title: 'Iterations'
    },
    vAxis: {
      title: 'Seconds'
    }
  }

  function drawCharts() {
    _drawCreatingInitialObjectChart();
    _drawFirstMethodCallChart();
    _drawSubsequentMethodCallsChart();
  }

  function _drawCreatingInitialObjectChart() {
    var data = google.visualization.arrayToDataTable([
      [ 'Iterations', 'Pure Hash', 'OpenStruct', 'Eager Loading', 'Method Missing', 'Lazy Loading' ],
      [          '0',    0.000007,     0.000002,        0.000002,         0.000003,       0.000002 ],
      [         '10',    0.000003,     0.001660,        0.000406,         0.000007,       0.000007 ],
      [        '100',    0.000008,     0.014470,        0.004800,         0.000029,       0.000029 ],
      [       '1000',    0.000052,     0.147943,        0.046868,         0.000269,       0.000535 ],
      [      '10000',    0.000514,     1.507701,        0.482254,         0.003883,       0.002656 ]
    ]);

    var chart = new google.visualization.LineChart(document.getElementById('chart_creating_initial_object'));
    chart.draw(data, options);
  }

  function _drawFirstMethodCallChart() {
    var data = google.visualization.arrayToDataTable([
      [ 'Iterations', 'Pure Hash', ' OpenStruct', 'Eager Loading', 'Method Missing', 'Lazy Loading' ],
      [          '0',    0.000008,      0.000002,        0.000003,         0.000002,       0.000002 ],
      [         '10',    0.000003,      0.001660,        0.000406,         0.000007,       0.000007 ],
      [        '100',    0.000236,      0.328859,        0.103952,         0.003961,       0.009337 ],
      [       '1000',    0.002248,      3.340419,        1.072228,         0.027261,       0.100895 ],
      [      '10000',    0.021093,     33.446652,       10.856142,         0.257472,       1.013682 ]
    ]);

    var chart = new google.visualization.LineChart(document.getElementById('chart_first_method_call'));
    chart.draw(data, options);
  }

  function _drawSubsequentMethodCallsChart() {
    var data = google.visualization.arrayToDataTable([
      [ 'Iterations', 'Pure Hash', ' OpenStruct', 'Eager Loading',  'Method Missing',  'Lazy Loading' ],
      [     '10,000',    0.021919,      0.078974,        0.084123,          0.186405,        0.072564 ],
      [    '100,000',    0.210670,      0.781603,        0.839965,          1.916927,        0.737112 ],
      [  '1,000,000',    2.155596,      7.977483,        8.513051,         19.288969,        7.400732 ],
      [ '10,000,000',   21.669286,     76.648551,       80.623932,        188.095461,       72.202056 ]
    ]);

    var chart = new google.visualization.LineChart(document.getElementById('subsequent_method_calls'));
    chart.draw(data, options);
  }
</script>
