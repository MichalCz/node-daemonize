About
=======

Node module for easy creation of daemons. Just write your daemon as plain 
node.js application (like `/examples/simple/app.js`) and wrap it with 
Daemonize (like `/examples/simple/ctrl.js`). 


Installation
==============
```
$ npm install daemonize
```


Example
=========

``` js
var daemon = require("daemonize").setup({
    main: "app.js",
    name: "sampleapp",
    pidfile: "sampleapp.pid"
});

switch (process.argv[2]) {
    
    case "start": 
        daemon.start().once("started", function() {
            process.exit();
        });
        break;
    
    case "stop":
        daemon.stop();
        break;
    
    default:
        console.log("Usage: [start|stop]");
}
```

For more examples see `examples` folder.

Documentation
===============

Daemonize works like standard `require()` but loaded module is 
shifted to work in background as a daemon. Keep in mind that
`stdin`, `stdout` and `stderr` are redirected to `/dev/null` 
so any output from daemon won't display in console. You need to 
use file for logging (ie like `/examples/advanced/app.js`).

Also any uncaught exception won't be displayed in the console,
so `process.on("uncaughtException", ...)` should be used to
redirect output to some log file.

## daemonize.setup(options)
Creates new `Daemon` instance. Supported `options`:
* `main` - main application module file to run as daemon (required)
* `name` - daemon name (default: basename of main)
* `pidfile` - pidfile path (default: `/var/run/[name].pid`)
* `user` - name or id of user (default: current)
* `group` - name or id of group (default: current)
* `silent` - disable printing info to console (default: `false`)
* `stopTimeout` - interval of daemon killing retry (default: `2s`)

All paths are resolved relative to file that uses "daemonize".

## Daemon
Daemon control class. It references daemon to control.

### Event: "starting"
`function() { }`

Emitted when `start()` is called and if daemon is not already running.

### Event: "started"
`function(pid) { }`

Emitted when daemon successfully started after calling `start()`.

### Event: "running"
`function(pid) { }`

Emitted when `start()` is called and a daemon is already running.

### Event: "stopping"
`function() { }`

Emitted when `stop()` or `kill()` is called and a daemon is running.

### Event: "stopped"
`function(pid) { }`

Emitted when daemon was successfully stopped after calling `stop()` 
or `kill()`.

### Event: "notrunning"
`function() { }`

Emitted when `stop()` or `kill()` is called and a deamon is not running.

### Event: "error"
`function(error) { }`

Emitted when `start()` failed. `error` is instance of `Error`. 
`error.message` contains information what went wrong.

### daemon.start()
Asynchronously start daemon. Emits `running` in case of daemon already 
running and `starting` when daemon is not running. Then emits `started`
when daemon successfully started. 

Emits `error` in case of any problem 
with starting daemon.

### daemon.stop()
Asynchronously stop daemon. Sends `SIGTERM` to daemon every 2s (or time 
set in options). 

Emits `notrunning` when daemon is not running, otherwise 
emits `stopping` and then `stopped` when daemon successfully stopped. 

### daemon.kill()
Asynchronously kill daemon. Sends `SIGTERM` and then (after 2s) `SIGKILL` 
to daemon. Repeats sending `SIGKILL` every 2s untill daemon stopps (time 
can be changed in options). 

Emits events same as `stop()`.

### daemon.status()
Synchronously returns pid for running daemon or `0` when daemon is not running.

### daemon.sendSignal(signal)
Synchronously sends `signal` to daemon and returns pid of daemon or `0` when
daemon is not running.


Changelog
===========

### 0.3.0 - Jan 29 2012
  - Daemon emits Events instead of console.log()
  - API change - events in place of callbacks

### 0.2.2 - Jan 27 2012
  - root priviledges no longer required
  - changed error exit codes
  - try to remove pidfile on daemon stop
  - configurable timeouts for start monitoring and killing
  - closing FD-s on daemon start
  - better examples

### 0.2.1 - Jan 26 2012
  - fix for calling callback in stop/kill when process is not running

### 0.2.0 - Jan 26 2012
  - code refactor
  - stop listening for uncaughtException
  - logfile removed

### 0.1.2 - Jan 25 2012
  - fixed stdout, stderr replacement
  - checking for daemon main module presence
  - signals change (added custom signals)
  - better log messages
  - gracefull terminate in example app
  - close logfile on process exit

### 0.1.1 - Jan 24 2012
  - print stacktrace for uncaughtException

### 0.1.0 - Jan 24 2012
  - First release 


License
=========

(The MIT License)

Copyright (c) 2012 Kuba Niegowski

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
