# Spark

Spark is a command-line tool used to manage node server processes written by [Tj Holowaychuk](http://github.com/visionmedia) and [Tim Caswell](http://github.com/creationix).  
It's part of the [Connect](http://github.com/senchalabs/connect) framework, however can be used standalone with _any_ node server.

This is a highly modified fork of the [original Spark](http://github.com/senchalabs/spark) library which adds a number of process management
and logging features &mdash; see the feature list below. This README documents the extra functionality that has been
added to this fork.  

Please report problems with any of the fork-specific features to [Alex Wolfe](http://github.com/alexkwolfe), 
rather than the Tj or Tim.  Fixes to the forked repository will be merged.

## Installation

Using the standard `npm` installation for Spark will get you the original library. This version is not hosted in the NPM package repository.
You must clone and install it locally:

    git clone git://github.com/alexkwolfe/spark.git
    cd spark
    npm install .

## Features

 - Port/Host to listen on
 - SSL hosting with specified key/certificate
 - Spawns a configurable number of worker processes
 - Environment modes (development/test/production)
 - **[this fork]** Server restart on file system changes
 - **[this fork]** Respawn worker processes gracefully on server restart
 - **[this fork]** Graceful or forcible server stop
 - **[this fork]** Server logging to ./logs/{environment mode}.js (configurable)
 - **[this fork]** Automatically loads configuration from ./config/{environment mode}.js (configurable)
 - User/Group to drop to after binding (if run as root)
 - Modify the node require paths
 - Can change the working directory before running the app
 - `--comment` option to label processes in system tools like `ps` or `htop`
 - Pass arbitrary code or environment variables to the app

## Making an app spark compatible

Any node server can be used with spark.  All you need to do it create a file called `app.js` that exports the instance of `http.Server` or `net.Server`.

A hello-world example would look like this:

    module.exports = require('http').createServer(function (req, res) {
        res.writeHead(200, {"Content-Type":"text-plain"});
        res.end("Hello World");
    });

And then to run it you simply go to the folder containing the `app.js` and type:

    $ spark

The output you'll see will be:

    Spark server(34037) listening on http://*:3000 in development mode

Where `34037` is the process id. If you want 4 processes to balance the requests across, no problem.

    $ spark -n 4

And you'll see:

    Spark server(34049) listening on http://*:3000 in development mode
    Spark server(34050) listening on http://*:3000 in development mode
    Spark server(34052) listening on http://*:3000 in development mode
    Spark server(34051) listening on http://*:3000 in development mode

## Commands

Spark supports starting, stopping, restarting, and checking the process status of your app. See
`spark -h` for more information.

The `spark restart` command is graceful. Configuration will be reloaded (some configuration 
changes such as host and port require a full stop/start). Each worker will be respawned after 
currently pending requests have completed. 

The `spark stop` command is also graceful. The server process will wait to shut down until
all currently pending requests have completed.  To forcibly stop the server without waiting
for requests to complete, use the `spark interrupt` command.

## Auto-restart on file changes

Spark will restart when the server's source files are changed on the file system. By default, 
all `.js` files in the current working directory will be watched. 

When running in an  environment other than "development" (i.e. `--env production`), auto-restart 
is disabled unless you specify the `--watchfile` option. The `--watchfile` option to specifies 
single file to watch &mdash; just touch the watched file to gracefully restart your app.

Auto-restart signals `HUP` in all environments except `development`, which always signals `INT`
on file system changes. That means that in development, any pending requests will be forcibly
terminated on file system changes.

## Logging

Spark exposes 'process.sparkEnv.log' to each server worker process. This logger writes to the
file specified in the `--logfile` option, which defaults to `./logs/{env.name}.log`. 

### Log levels
You can use the `log.debug`, `log.info`, and `log.error` functions to write to the configured file. Use the
`--loglevel` option to control the verbosity of your application's log output. For example, setting 
`--loglevel info` will log 'info', and 'error' log messages, but not 'debug' messages. The log level
uses sensible defaults based on `--env`.

### Sending an Error to the logger

Logging an `Error` object will automatically log it as an 'error' and will include the stack trace.

### Writing to output

When you use the `--verbose` option, the Spark process itself will write to STDERR, and will also
direct your apps log statements to STDOUT (while adhering to the configured log level).

### Customizing log statement format

To customize the date format that the logger uses, replace the `log.formatDate(date)` function. It
takes a single `Date` parameter and returns a `String`.

To customize the format of the log statement, replace the `log.format(date, level, msg)` function with
your own function that returns a `String`.

### Logging options in the configuration file

All logging configuration goes under the `log` key in your config file:

    module.exports = {
	   host: 'localhost',
	   port: 80,
	   log : { 
		 level : 'warn', 
		 file  : './logs/applog.log'
	   }
    }

### Log rotation

Spark does not automatically rotate your log files &mdash; logrotate is recommended. Make sure to use 
the `copytruncate` option, so that your app can continue writing to the same log file:

	/var/www/vhosts/your_project/logs/*log {
	   daily
	   rotate 7
	   missingok
	   copytruncate
	}


## MIT License

Copyright (c) 2010 Sencha Inc.

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
