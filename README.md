# Async.js

fork of [caolan/async](https://github.com/caolan/async) that just includes
priority queue

## Documentation

### Control Flow

* [`queue`](#queue)
* [`priorityQueue`](#priorityQueue)
* [`nextTick`](#nextTick)

### Utils

* [`noConflict`](#noConflict)


## Control Flow

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
