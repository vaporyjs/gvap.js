gvap.js
=======

[![Build Status](https://travis-ci.org/vaporyjs/gvap.js.svg)](https://travis-ci.org/vaporyjs/gvap.js)
[![Coverage Status](https://coveralls.io/repos/vaporyjs/gvap.js/badge.svg?branch=master&service=github)](https://coveralls.io/github/vaporyjs/gvap.js?branch=master)
[![npm version](https://badge.fury.io/js/gvap.svg)](https://badge.fury.io/js/gvap)

Start and stop [gvap](https://github.com/vaporyco/go-vapory) from Node.js.

Usage
-----

```
$ npm install gvap
```
To use gvap.js, simply require it:
```javascript
var gvap = require("gvap");
```

### Starting and stopping gvap

gvap's `start` method accepts a configuration object, which uses the same flags as the gvap command line client.  (Here, the flags are organized into an object.)  Flags that are not accompanied by a value on the command line (for example, `--mine`) should be passed in as `{ flag: null }`.
```javascript
var options = {
    networkid: "10101",
    port: 30303,
    rpcport: 8545,
    mine: null
};

gvap.start(options, function (err, proc) {
    if (err) return console.error(err);
    // get your gvap on!
});
```
The callback's parameter `proc` is the child process, which is also attached to the `gvap` object (`gvap.proc`) for your convenience.

When you and gvap have had enough lively times, `stop` kills the underlying gvap process:
```javascript
gvap.stop(function () {
    // no more lively times :( 
});
```

### Listeners

The callback for `start` fires after gvap has successfully started.  Specifically, it looks for `"IPC service started"` in gvap's stderr.  If you want to change the start callback trigger to something else, this can be done by replacing gvap's default listeners.  `gvap.start` accepts an optional second parameter which should specify the listeners to overwrite, for example:
```javascript
{
    stdout: function (data) {
        process.stdout.write("I got a message!! " + data.toString());
    },
    stderr: function (data) {
        if (data.toString().indexOf("Protocol Versions") > -1) {
            gvap.trigger(null, gvap.proc);
        }
    },
    close: function (code) {
        console.log("It's game over, man!  Game over!");
    }
}
```
In the above code, `gvap.trigger` is the callback passed to `gvap.start`.  (This callback is stored so that the startup trigger can be changed if needed.)  These three listeners (`stdout`, `stderr`, and `close`) are the only listeners which can be specified in this parameter, since only these three are set by default in `gvap.start`.

If you want to swap out or add other listeners (after the initial startup), you can use the `gvap.listen` convenience method:
```javascript
gvap.listen("stdout", "data", function (data) { process.stdout.write(data); });
```
This example (re-)sets the "data" listener on stdout by setting `gvap.proc.stdout._events.data = function (data) { process.stdout.write(data); }`.

Tests
-----

gvap.js's tests use [vaprpc](https://github.com/AugurProject/vaprpc) to send some basic requests to gvap.
```
$ npm test
```
