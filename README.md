# intrvl

A lightweight utility for creating a node.js interval that stops itself based on timeout and/or max execution count as well as event emission.

### Installation

```
npm install intrvl
```

### Usage

#### intrvl.start(toExecute, intervalMillis, timeoutMillis, maxExecutionCount)
* `toExecute` Function. Executes at every interval
* `intervalMillis` Number. How many milliseconds between execution
* `timeoutMillis` Number. How long to allow the interval to run. Left `undefined`, the interval will run without a timeout
* `maxExecutionCount` Number. How many times to execute the interval. Left `undefined`, the interval will run without a max

Returns a new `Intrvl` class instance (running by default)

#### Class: intrvl.Intrvl
Passed back from the `intrvl.start` method.

##### Intrvl.stop()
Causes the interval to stop execution, as if a timeout or max had been reached.
```
var intrvl = require('intrvl');

var interval = intrvl.start(function(){
    console.log('Hello, World')
}, 1000); // runs indefinitely

interval.on('stop', function(execCount) {
    console.log('Stopped after ' + execCount + ' executions');
});

// ... do some other logic, answer some server requests, etc..

// ... oh no, I need to cancel the interval!
interval.stop();
```

##### Event: 'exec'
Emitted on each execution of the interval. Argument is the current execution count
```
var intrvl = require('intrvl');

var interval = intrvl.start(function(){
    console.log('Hello, World');
}, 1000, undefined, 5); // executes every second, stops after 5 executions

interval.on('exec', function(execCount) {
    console.log('Execution ' + execCount + ' occured');
});
```

##### Event: 'stop'
Emitted when an interval has finished, either by self-imposed limits (timeoutMills, maxExecutionCount), or by calling `.stop()` on a running instance. Argument is the total execution count

```
var intrvl = require('intrvl');

var interval = intrvl.start(function(){
    console.log('Hello, World');
}, 1000, undefined, 5); // executes every second, stops after 5 executions

interval.on('stop', function(execCount) {
    console.log('Executions: ' + execCount);
});
```

### More Examples
Using competing configurations
```
// runs every 1 ms, ends after 100ms or 100 executions, whichever happens first!

var intrvl = require('intrvl');

var intervalMillis = 1;
var timeoutMillis = 100;
var maxExecutionCount = 100;

var interval = intrvl.start(function() {
    console.log('Hello, World');
}, intervalMillis, timeoutMillis, maxExecutionCount);

interval.on('stop', function(execCount) {
    console.log('ENDED: ' + execCount);
});
```

Inifinite intrvl still gives us events!
```
// runs every 1 ms, never ends
// might as well use setInterval at this point, right?
// hold up - we can still have fun with events!

var intrvl = require('intrvl');

var intervalMillis = 1;
var timeoutMillis = undefined;
var maxExecutionCount = undefined;

var interval = intrvl.start(function() {
    console.log('Hello, World');
}, intervalMillis, timeoutMillis, maxExecutionCount);

interval.on('exec', function(execCount) {
    console.log('Execution count is growing, currently at ' + execCount);
});

interval.on('stop', function(execCount) {
    console.log('I\'m never-ending, unless someone else tells me otherwise');
});
```
