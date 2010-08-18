# Spark

Spark is a command-line tool used to manage node server processes written by [Tj Holowaychuk](http://github.com/visionmedia) and [Tim Caswell](http://github.com/creationix).  
It's part of the [Connect](http://github.com/senchalabs/connect) framework, however can be used standalone with _any_ node server.

## Features

 - Port/Host to listen on
 - SSL hosting with specified key/certificate
 - Automatic child process spawning for multi-process node servers that share a common port
 - Respawn child processes gracefully on server restart
 - Graceful or forcible server stop
 - Automatic server restart when file system changes
 - User/Group to drop to after binding (if run as root)
 - Environment modes (development/testing/production)
 - Modify the node require paths
 - Can load any of these configs from from a file (optionally grouped by env)
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
