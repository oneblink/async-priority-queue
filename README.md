# Async.js

fork of [caolan/async](https://github.com/caolan/async) that just includes
priority queue

## Documentation

### Collections

* [`each`](#each)
* [`eachSeries`](#eachSeries)
* [`map`](#map)
* [`mapSeries`](#mapSeries)
* [`concat`](#concat)
* [`concatSeries`](#concatSeries)

### Control Flow

* [`series`](#seriestasks-callback)
* [`parallel`](#parallel)
* [`queue`](#queue)
* [`priorityQueue`](#priorityQueue)
* [`retry`](#retry)
* [`apply`](#apply)
* [`nextTick`](#nextTick)

### Utils

* [`noConflict`](#noConflict)


## Collections

<a name="forEach" />
<a name="each" />
### each(arr, iterator, callback)

Applies the function `iterator` to each item in `arr`, in parallel.
The `iterator` is called with an item from the list, and a callback for when it
has finished. If the `iterator` passes an error to its `callback`, the main
`callback` (for the `each` function) is immediately called with the error.

Note, that since this function applies `iterator` to each item in parallel,
there is no guarantee that the iterator functions will complete in order.

__Arguments__

* `arr` - An array to iterate over.
* `iterator(item, callback)` - A function to apply to each item in `arr`.
  The iterator is passed a `callback(err)` which must be called once it has
  completed. If no error has occurred, the `callback` should be run without
  arguments or with an explicit `null` argument.
* `callback(err)` - A callback which is called when all `iterator` functions
  have finished, or an error occurs.

__Examples__


```js
// assuming openFiles is an array of file names and saveFile is a function
// to save the modified contents of that file:

async.each(openFiles, saveFile, function(err){
    // if any of the saves produced an error, err would equal that error
});
```

```js
// assuming openFiles is an array of file names

async.each(openFiles, function(file, callback) {

  // Perform operation on file here.
  console.log('Processing file ' + file);

  if( file.length > 32 ) {
    console.log('This file name is too long');
    callback('File name too long');
  } else {
    // Do work to process file here
    console.log('File processed');
    callback();
  }
}, function(err){
    // if any of the file processing produced an error, err would equal that error
    if( err ) {
      // One of the iterations produced an error.
      // All processing will now stop.
      console.log('A file failed to process');
    } else {
      console.log('All files have been processed successfully');
    }
});
```

---------------------------------------

<a name="forEachSeries" />
<a name="eachSeries" />
### eachSeries(arr, iterator, callback)

The same as [`each`](#each), only `iterator` is applied to each item in `arr` in
series. The next `iterator` is only called once the current one has completed.
This means the `iterator` functions will complete in order.


---------------------------------------

<a name="map" />
### map(arr, iterator, callback)

Produces a new array of values by mapping each value in `arr` through
the `iterator` function. The `iterator` is called with an item from `arr` and a
callback for when it has finished processing. Each of these callback takes 2 arguments:
an `error`, and the transformed item from `arr`. If `iterator` passes an error to his
callback, the main `callback` (for the `map` function) is immediately called with the error.

Note, that since this function applies the `iterator` to each item in parallel,
there is no guarantee that the `iterator` functions will complete in order.
However, the results array will be in the same order as the original `arr`.

__Arguments__

* `arr` - An array to iterate over.
* `iterator(item, callback)` - A function to apply to each item in `arr`.
  The iterator is passed a `callback(err, transformed)` which must be called once
  it has completed with an error (which can be `null`) and a transformed item.
* `callback(err, results)` - A callback which is called when all `iterator`
  functions have finished, or an error occurs. Results is an array of the
  transformed items from the `arr`.

__Example__

```js
async.map(['file1','file2','file3'], fs.stat, function(err, results){
    // results is now an array of stats for each file
});
```

---------------------------------------

<a name="mapSeries" />
### mapSeries(arr, iterator, callback)

The same as [`map`](#map), only the `iterator` is applied to each item in `arr` in
series. The next `iterator` is only called once the current one has completed.
The results array will be in the same order as the original.



## Control Flow

<a name="series" />
### series(tasks, [callback])

Run the functions in the `tasks` array in series, each one running once the previous
function has completed. If any functions in the series pass an error to its
callback, no more functions are run, and `callback` is immediately called with the value of the error.
Otherwise, `callback` receives an array of results when `tasks` have completed.

It is also possible to use an object instead of an array. Each property will be
run as a function, and the results will be passed to the final `callback` as an object
instead of an array. This can be a more readable way of handling results from
[`series`](#series).

**Note** that while many implementations preserve the order of object properties, the
[ECMAScript Language Specifcation](http://www.ecma-international.org/ecma-262/5.1/#sec-8.6)
explicitly states that

> The mechanics and order of enumerating the properties is not specified.

So if you rely on the order in which your series of functions are executed, and want
this to work on all platforms, consider using an array.

__Arguments__

* `tasks` - An array or object containing functions to run, each function is passed
  a `callback(err, result)` it must call on completion with an error `err` (which can
  be `null`) and an optional `result` value.
* `callback(err, results)` - An optional callback to run once all the functions
  have completed. This function gets a results array (or object) containing all
  the result arguments passed to the `task` callbacks.

__Example__

```js
async.series([
    function(callback){
        // do some stuff ...
        callback(null, 'one');
    },
    function(callback){
        // do some more stuff ...
        callback(null, 'two');
    }
],
// optional callback
function(err, results){
    // results is now equal to ['one', 'two']
});


// an example using an object instead of an array
async.series({
    one: function(callback){
        setTimeout(function(){
            callback(null, 1);
        }, 200);
    },
    two: function(callback){
        setTimeout(function(){
            callback(null, 2);
        }, 100);
    }
},
function(err, results) {
    // results is now equal to: {one: 1, two: 2}
});
```

---------------------------------------

<a name="parallel" />
### parallel(tasks, [callback])

Run the `tasks` array of functions in parallel, without waiting until the previous
function has completed. If any of the functions pass an error to its
callback, the main `callback` is immediately called with the value of the error.
Once the `tasks` have completed, the results are passed to the final `callback` as an
array.

It is also possible to use an object instead of an array. Each property will be
run as a function and the results will be passed to the final `callback` as an object
instead of an array. This can be a more readable way of handling results from
[`parallel`](#parallel).


__Arguments__

* `tasks` - An array or object containing functions to run. Each function is passed
  a `callback(err, result)` which it must call on completion with an error `err`
  (which can be `null`) and an optional `result` value.
* `callback(err, results)` - An optional callback to run once all the functions
  have completed. This function gets a results array (or object) containing all
  the result arguments passed to the task callbacks.

__Example__

```js
async.parallel([
    function(callback){
        setTimeout(function(){
            callback(null, 'one');
        }, 200);
    },
    function(callback){
        setTimeout(function(){
            callback(null, 'two');
        }, 100);
    }
],
// optional callback
function(err, results){
    // the results array will equal ['one','two'] even though
    // the second function had a shorter timeout.
});


// an example using an object instead of an array
async.parallel({
    one: function(callback){
        setTimeout(function(){
            callback(null, 1);
        }, 200);
    },
    two: function(callback){
        setTimeout(function(){
            callback(null, 2);
        }, 100);
    }
},
function(err, results) {
    // results is now equals to: {one: 1, two: 2}
});
```

---------------------------------------

<a name="queue" />
### queue(worker, concurrency)

Creates a `queue` object with the specified `concurrency`. Tasks added to the
`queue` are processed in parallel (up to the `concurrency` limit). If all
`worker`s are in progress, the task is queued until one becomes available.
Once a `worker` completes a `task`, that `task`'s callback is called.

__Arguments__

* `worker(task, callback)` - An asynchronous function for processing a queued
  task, which must call its `callback(err)` argument when finished, with an
  optional `error` as an argument.
* `concurrency` - An `integer` for determining how many `worker` functions should be
  run in parallel.

__Queue objects__

The `queue` object returned by this function has the following properties and
methods:

* `length()` - a function returning the number of items waiting to be processed.
* `started` - a function returning whether or not any items have been pushed and processed by the queue
* `running()` - a function returning the number of items currently being processed.
* `idle()` - a function returning false if there are items waiting or being processed, or true if not.
* `concurrency` - an integer for determining how many `worker` functions should be
  run in parallel. This property can be changed after a `queue` is created to
  alter the concurrency on-the-fly.
* `push(task, [callback])` - add a new task to the `queue`. Calls `callback` once
  the `worker` has finished processing the task. Instead of a single task, a `tasks` array
  can be submitted. The respective callback is used for every task in the list.
* `unshift(task, [callback])` - add a new task to the front of the `queue`.
* `saturated` - a callback that is called when the `queue` length hits the `concurrency` limit,
   and further tasks will be queued.
* `empty` - a callback that is called when the last item from the `queue` is given to a `worker`.
* `drain` - a callback that is called when the last item from the `queue` has returned from the `worker`.
* `paused` - a boolean for determining whether the queue is in a paused state
* `pause()` - a function that pauses the processing of tasks until `resume()` is called.
* `resume()` - a function that resumes the processing of queued tasks when the queue is paused.
* `kill()` - a function that removes the `drain` callback and empties remaining tasks from the queue forcing it to go idle.

__Example__

```js
// create a queue object with concurrency 2

var q = async.queue(function (task, callback) {
    console.log('hello ' + task.name);
    callback();
}, 2);


// assign a callback
q.drain = function() {
    console.log('all items have been processed');
}

// add some items to the queue

q.push({name: 'foo'}, function (err) {
    console.log('finished processing foo');
});
q.push({name: 'bar'}, function (err) {
    console.log('finished processing bar');
});

// add some items to the queue (batch-wise)

q.push([{name: 'baz'},{name: 'bay'},{name: 'bax'}], function (err) {
    console.log('finished processing item');
});

// add some items to the front of the queue

q.unshift({name: 'bar'}, function (err) {
    console.log('finished processing bar');
});
```


---------------------------------------

<a name="priorityQueue" />
### priorityQueue(worker, concurrency)

The same as [`queue`](#queue) only tasks are assigned a priority and completed in ascending priority order. There are two differences between `queue` and `priorityQueue` objects:

* `push(task, priority, [callback])` - `priority` should be a number. If an array of
  `tasks` is given, all tasks will be assigned the same priority.
* The `unshift` method was removed.

---------------------------------------

<a name="retry" />
### retry([times = 5], task, [callback])

Attempts to get a successful response from `task` no more than `times` times before
returning an error. If the task is successful, the `callback` will be passed the result
of the successful task. If all attempts fail, the callback will be passed the error and
result (if any) of the final attempt.

__Arguments__

* `times` - An integer indicating how many times to attempt the `task` before giving up. Defaults to 5.
* `task(callback, results)` - A function which receives two arguments: (1) a `callback(err, result)`
  which must be called when finished, passing `err` (which can be `null`) and the `result` of
  the function's execution, and (2) a `results` object, containing the results of
  the previously executed functions (if nested inside another control flow).
* `callback(err, results)` - An optional callback which is called when the
  task has succeeded, or after the final failed attempt. It receives the `err` and `result` arguments of the last attempt at completing the `task`.

The [`retry`](#retry) function can be used as a stand-alone control flow by passing a
callback, as shown below:

```js
async.retry(3, apiMethod, function(err, result) {
    // do something with the result
});
```

It can also be embeded within other control flow functions to retry individual methods
that are not as reliable, like this:

```js
async.auto({
    users: api.getUsers.bind(api),
    payments: async.retry(3, api.getPayments.bind(api))
}, function(err, results) {
  // do something with the results
});
```


---------------------------------------

<a name="apply" />
### apply(function, arguments..)

Creates a continuation function with some arguments already applied.

Useful as a shorthand when combined with other control flow functions. Any arguments
passed to the returned function are added to the arguments originally passed
to apply.

__Arguments__

* `function` - The function you want to eventually apply all arguments to.
* `arguments...` - Any number of arguments to automatically apply when the
  continuation is called.

__Example__

```js
// using apply

async.parallel([
    async.apply(fs.writeFile, 'testfile1', 'test1'),
    async.apply(fs.writeFile, 'testfile2', 'test2'),
]);


// the same process without using apply

async.parallel([
    function(callback){
        fs.writeFile('testfile1', 'test1', callback);
    },
    function(callback){
        fs.writeFile('testfile2', 'test2', callback);
    }
]);
```

It's possible to pass any number of additional arguments when calling the
continuation:

```js
node> var fn = async.apply(sys.puts, 'one');
node> fn('two', 'three');
one
two
three
```

---------------------------------------

<a name="nextTick" />
### nextTick(callback), setImmediate(callback)

Calls `callback` on a later loop around the event loop. In Node.js this just
calls `process.nextTick`; in the browser it falls back to `setImmediate(callback)`
if available, otherwise `setTimeout(callback, 0)`, which means other higher priority
events may precede the execution of `callback`.

This is used internally for browser-compatibility purposes.

__Arguments__

* `callback` - The function to call on a later loop around the event loop.

__Example__

```js
var call_order = [];
async.nextTick(function(){
    call_order.push('two');
    // call_order now equals ['one','two']
});
call_order.push('one')
```


## Utils

<a name="noConflict" />
### noConflict()

Changes the value of `async` back to its original value, returning a reference to the
`async` object.
