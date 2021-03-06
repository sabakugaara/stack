# Stack

Software Development Kit and Command Line Interface for spawning streaming HTTP [microservices](http://martinfowler.com/articles/microservices.html) in multiple programming languages.

At it's core, `stack` maps HTTP request/response streams to the STDIN/STDOUT streams of any programming language binary.

## Introduction

This project is the component which several production services, including [hook.io](http://hook.io), use to spawn real-time arbitrary streaming microservices in response to streaming HTTP requests. It's been battle-hardened with over two years of development and 100+ million microservice deployments.

You are encouraged to use this module as-is, or modify it to suite your needs. If you are interested in contributing please let us know by opening a Pull Request.

## Features

 - Creates HTTP microservices in multiple Programming Languages
 - Ships with `stack` binary for starting HTTP microservice servers
 - Maps HTTP request / response to STDIN / STDOUT of spawned child processes
 - Uses a "system process per microservice request" design
 - Isolates state of microservice per system process and request ( stateless service requests )
 - Microservice error handling and custom timeouts
 - Can parse any kind of request data
   - Query
   - JSON
   - Form
   - Multipart
   - Streaming
   - Binary

## Installation

    npm install -g stackvana

*This will install the `stack` binary globally on your system.*

## Usage

```
  Usage: stack [command] [options]

  Commands:

    help  Display help

  Options:

    -t, --timeout <number>  Sets max timeout of service in milliseconds
    -h, --host <value>      Host to listen on
    -p, --port <number>     Port to listen on
    -l, --language <value>  Target programming languages
    -w, --watch <bool>      Reloads source files on every request ( dev only )
    -v, --version           Output the version number
```

By default, `stack` will attempt to start a listening HTTP server based on a microservice file path.

### As Command Line Interface

    stack ./path/to/script.foo

#### CLI Examples

    stack ./examples/services/echo/echo.js
    stack -l babel ./examples/services/echo/echo-es6-async.js
    stack ./examples/services/echo/echo.sh
    stack ./examples/services/echo/echo.lua
    stack ./examples/services/echo/echo.php
    stack ./examples/services/echo/echo.pl
    stack ./examples/services/echo/echo.py
    stack -l python3 ./examples/services/echo/echo-py3.py
    stack ./examples/services/echo/echo.rb
    stack ./examples/services/echo/echo.coffee
    stack ./examples/services/echo/echo.ss
    stack ./examples/services/echo/echo.st
    stack ./examples/services/echo/echo.tcl

Each call to `stack` will automatically start a listening HTTP server on port `3000`, additional instances of `stack` will auto-increment the port to `3001`, `3002`, etc.

Service target language is automatically detected based on the file extension of the service. This can be overridden using the `--language` option. 

*Note: For certain languages ( such as Babel ), the first microservice request to `stack` may take additional time as it will perform an initial compile and cache step.*

*Note: Please see [Babel Support](#babel) for additional Babel configuration*

### Programmatically Inside Node.js

```
Node API

spawn(opts, req, res)

   opts.code      - source code of microservice
   opts.language  - target programming language
   opts.log       - optional custom logging handler function ( defaults to `console.log` )
   opts.env       - environment variables which populates `service.env` ( defaults to `process.env` )

   req - http request stream
   res - http response stream

```

``` 
Hook Object API

   Every service is populated with an object named "hook"
   Note: This API may differ slightly per language implementation, see ./examples/services

   service.params - combined scope of all incoming HTTP request variables
   service.env    - environment variables of service ( defaults to `process.env`)
   service.req    - HTTP Request stream
   service.res    - HTTP Response stream

```

### Example services

see: `./examples/services` for more details

```js
var stack = require('stackvana');
var http = require('http');

var bashService = 'echo "hello bash"';
var pythonService = 'print "hello python"';
var nodeService = function testService (opts) {
  var res = opts.res;
  res.write('hello node!');
  res.end();
};

var server = http.createServer(function(req, res){
  stack.spawn({
    code: nodeService,
    language: "javascript"
  }, req, res);
  // or you could use bash / any other supported language
  // stack.spawn({code: bashService, language: "bash" }, req, res);
  // stack.spawn({code: pythonService, language: "python" }, req, res);
  
});
server.listen(3000, function () {
  console.log('http server started on port 3000');
});

```

## Multiple Microservices Per Server Instance

In some configurations you may want to safetly run multiple kinds of microservices on one server instance ( a small monolith ). `stack` is designed exactly for this use case.

Since every incoming service request will spawn a separate process, `stack` can safely and easily handle spawning multiple types of microservices at once without affecting the state of other services.

If you look at the `./examples/simple-http-server` file, you will see that `spawn()` can be used as a standard Node.js or Express HTTP middleware. For multiple services per server, simply map the `spawn()` method to any custom routes you may want to define.

### Supports Microservices In Many Languages

  - javascript
  - babel ( ES6 / ES7 / etc ... )
  - coffee-script
  - bash
  - lua
  - perl
  - php
  - python
  - python3
  - ruby
  - scheme
  - smalltalk
  - tcl

*Additional language support is both planned and welcomed. Please open a Pull Request if you wish to see a specific language added*

<a name="babel"></a>

## Babel

In order to run Babel / ES6 / ES7 microservices, you must install the following packages:

```
npm install babel-core
npm install babel-plugin-syntax-async-functions
npm install babel-plugin-transform-regenerator
npm install babel-polyfill
npm install babel-preset-es2015
npm install babel-preset-stage-3
npm install then-sleep
```

## Security

Running untrusted microservice code in a safe way is a complex problem. The `stack` module is only intended to isolate a small part of the entire untrusted source code execution chain.

**If you intend to use this module to run untrusted source code please consider the following:**

### What this module does isolate

 - Microservice state
 - Microservice errors
 - Stream / Socket errors
 - HTTP Request state

Every incoming HTTP request will spawn a new system process to run the microservice instance, which then sends data to the HTTP response. Every microservice execution will happen in a fresh systems process.

All errors that can possibly happen during the execution of a microservice should be trapped and isolated to not affect any other microservices.

### What this mode does **NOT** isolate

 - Server Memory
 - Server CPU
 - Server file-system
 - Process CPU
 - Process Memory

`stack` cannot make any guarantees about the isolation of the server or spawned processes.  All microservices will have default access to the server's file-system and child processes.

To ensure isolation of the server file-system, you would want to use the `stack` binary in a `chroot` jail, or another similar container solution.

To ensure isolation of the server memory and cpu, you will want to use the `stack` binary in a virtualized environment capable of monitoring and managing resource usage per process.

### Credits

Author: 

 - [Marak Squires](https://github.com/marak)

Special thanks to:

 - [Jordan Harband](https://github.com/ljharb)
 - [Andrew Love](https://github.com/andrewtlove) 

for their feedback in helping review early versions of this project.
