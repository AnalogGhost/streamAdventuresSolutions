#**Questions?** Ask them [here](http://goo.gl/forms/DnUb9CvwYmJMKUc33)!

# Stream Adventures

## 1.BEEP BOOP

### Problem

Just to make sure everything is working, just write a `program.js` that outputs
the string "beep boop" with a `console.log()`.

To verify your `program.js` against the expected output, do:

  stream-adventure verify program.js

for more options, run `stream-adventure help`.

### Solution

This one is fairly straightforward.  Just add a console.log that outputs the string "beep boop".

```javascript
console.log('beep boop');
```
## 2.MEET PIPE
### Problem

You will get a file as the first argument to your program (process.argv[2]).

Use `fs.createReadStream()` to pipe the given file to `process.stdout`.

`fs.createReadStream()` takes a file as an argument and returns a readable
stream that you can call `.pipe()` on. Here's a readable stream that pipes its
data to `process.stderr`:

    var fs = require('fs');
    fs.createReadStream('data.txt').pipe(process.stderr);

Your program is basically the same idea, but instead of `'data.txt'`, the
filename comes from `process.argv[2]` and you should pipe to stdout, not stderr.

### Solution

```javascript
var fs = require('fs');
var file = process.argv[2];
fs.createReadStream(file).pipe(process.stdout);
```

The first change from the example given is getting the filename from `process.argv[2]`.  We created a variable `file` and assign it the value of `process.argv[2]`.

The `process.argv` property returns an array containing the command line arguments passed when the Node.js process was launched. [Would you like to know more?](https://nodejs.org/docs/latest/api/process.html#process_process_argv)

The second change is from the example is to pipe the data to stdout instead of stderr.

The `process.stdout` propety returns a Writable stream equivalent to or associated with stdout. [Would you like to know more?](https://nodejs.org/docs/latest/api/process.html#process_process_stdout)

## 3.INPUT OUTPUT

### Problem
Take data from `process.stdin` and pipe it to `process.stdout`.

With `.pipe()`. `process.stdin.pipe()` to be exact.

Don't overthink this.

### Solution
```javascript
process.stdin.pipe(process.stdout);
```

Pretty much as the instructions explain.  Don't overthink it.

## 4.TRANSFORM

### Problem

Convert data from `process.stdin` to upper-case data on `process.stdout` using the `through2` module.

To get the `through2` module you'll need to do:

    npm install through2

A transform stream takes input data and applies an operation to the data to produce the output data.

Create a through stream with a `write` and `end` function:

    var through = require('through2');
    var stream = through(write, end);

The `write` function is called for every buffer of available input:

    function write (buffer, encoding, next) {
        // ...
    }

and the `end` function is called when there is no more data:

    function end () {
        // ...
    }

Inside the write function, call `this.push()` to produce output data and call `next()` when you're ready to receive the next chunk:

    function write (buffer, encoding, next) {
        this.push('I got some data: ' + buffer + '\n');
        next();
    }

and call `done()` to finish the output:

    function end (done) {
        done();
    }

`write` and `end` are both optional.

If `write` is not specified, the default implementation passes the input data to the output unmodified.

If `end` is not specified, the default implementation calls `this.push(null)` to close the output side when the input side ends.

Make sure to pipe `process.stdin` into your transform stream
and pipe your transform stream into `process.stdout`, like this:

    process.stdin.pipe(tr).pipe(process.stdout);

To convert a buffer to a string, call `buffer.toString()`.

### Solution
```javascript
var through = require('through2');
var stream = through(function (buf, encoding, next) {
    this.push(buf.toString().toUpperCase());
    next();
});
process.stdin.pipe(stream).pipe(process.stdout);
```

First we have to require the `through2` module.

The next line uses the through function.  The first parameter is the `write` function - here we are creating it inline, but you can also declare it outside and pass it in.  

We didn't pass in an `end` function since its optional and the default implementation calls `this.push(null)` to close the output side when the input side ends.  Thats exactly what we want anyway.

Finally we take `process.stdin` and `pipe` it to `stream` (which we created on the second line) and then `pipe` to `stdout`.

## 5.LINES

### Problem

Instead of transforming every line as in the previous "TRANSFORM" example, for this challenge, convert even-numbered lines to upper-case and odd-numbered lines to lower-case. Consider the first line to be odd-numbered. For example given this input:

    One
    Two
    Three
    Four

Your program should output:

    one
    TWO
    three
    FOUR

You can use the `split` module to split input by newlines. For example:

    var split = require('split');
    process.stdin
        .pipe(split())
        .pipe(through2(function (line, _, next) {
            console.dir(line.toString());
            next();
        }))
    ;

`split` will buffer chunks on newlines before you get them. In the previous
example, we will get separate events for each line even though all the data
probably arrives on the same chunk:

    $ echo -e 'one\ntwo\nthree' | node split.js
    'one'
    'two'
    'three'

Your own program should use `split` in this way, but you should transform the
input and pipe the output through to `process.stdout`.

Make sure to `npm install split through2` in the directory where your solution
file lives.

### Solution

```javascript
var through = require('through2');
var split = require('split');

var lineCount = 0;
var tr = through(function (buf, _, next) {
    var line = buf.toString();
    this.push(lineCount % 2 === 0
        ? line.toLowerCase() + '\n'
        : line.toUpperCase() + '\n'
    );
    lineCount ++;
    next();
});

process.stdin
    .pipe(split())
    .pipe(tr)
    .pipe(process.stdout);
```


Some help for this method.  We keep track of the current line being processed with the variable linecount.  

We use a turnary to either uppercase or lowercase based on lineCount being even or odd.

So we take `process.stdin` and pipe it to `split`.  `split` is going to break the input on newlines.  Then we take each of those lines and pipe it to our `tr` function.  `tr` will handle the uppercase/lowercase based on which line we are on.  We then pipe that to `process.stdout`.

## 6.CONCAT
###Problem
You will be given text on process.stdin. Buffer the text and reverse it using
the `concat-stream` module before writing it to stdout.

`concat-stream` is a write stream that you can pass a callback to get the
complete contents of a stream as a single buffer. Here's an example that uses
concat to buffer POST content in order to JSON.parse() the submitted data:

    var concat = require('concat-stream');
    var http = require('http');
    
    var server = http.createServer(function (req, res) {
        if (req.method === 'POST') {
            req.pipe(concat(function (body) {
                var obj = JSON.parse(body);
                res.end(Object.keys(obj).join('\n'));
            }));
        }
        else res.end();
    });
    server.listen(5000);

In your adventure you'll only need to buffer input with `concat()` from
process.stdin.

Make sure to `npm install concat-stream` in the directory where your solution
file is located.

### Solution

```javascript
var concat = require('concat-stream');
  
process.stdin.pipe(concat(function (src) {
  var s = src.toString().split('').reverse().join('');
  console.log(s);
}));
```

First we require the module `concat-stream`.  Then we take `process.stdin` and pipe it to the `concat` function.  The `concat` module will take all the contents of a stream and treat it as a single buffer (as opposed to one chunk at a time).  We pass a function into `concat` that converts the buffer to a string, splits it into an array, reverses the array, then joins the array - creating a string again.  This basically reverses our string.

We then console.log the result.

## 7.HTTP SERVER

### Problem
In this challenge, write an http server that uses a through stream to write back
the request stream as upper-cased response data for POST requests.

Streams aren't just for text files and stdin/stdout. Did you know that http
request and response objects from node core's `http.createServer()` handler are
also streams?

For example, we can stream a file to the response object:

    var http = require('http');
    var fs = require('fs');
    var server = http.createServer(function (req, res) {
        fs.createReadStream('file.txt').pipe(res);
    });
    server.listen(process.argv[2]);

This is great because our server can respond immediately without buffering
everything in memory first.

We can also stream a request to populate a file with data:

    var http = require('http');
    var fs = require('fs');
    var server = http.createServer(function (req, res) {
        if (req.method === 'POST') {
            req.pipe(fs.createWriteStream('post.txt'));
        }
        res.end('beep boop\n');
    });
    server.listen(process.argv[2]);

You can test this post server with curl:

    $ node server.js 8000 &
    $ echo hack the planet | curl -d@- http://localhost:8000
    beep boop
    $ cat post.txt
    hack the planet

Your http server should listen on the port given at process.argv[2] and convert
the POST request written to it to upper-case using the same approach as the
TRANSFORM example.

As a refresher, here's an example with the default through2 callbacks explicitly
defined:

    var through = require('through2');
    process.stdin.pipe(through(write, end)).pipe(process.stdout);

    function write (buf, _, next) {
      this.push(buf);
      next();
    }
    function end (done) { done(); }

Do that, but send upper-case data in your http server in response to POST data.

Make sure to `npm install through2` in the directory where your solution file
lives.

###Solution
```
var http = require('http');
var through = require('through2');
  
var server = http.createServer(function (req, res) {
  if (req.method === 'POST') {
      req.pipe(through(function (buf, _, next) {
          this.push(buf.toString().toUpperCase());
          next();
      })).pipe(res);
  }
  else res.end('send me a POST\n');
});
server.listen(parseInt(process.argv[2]));
```

Include the `http` and `through2` modules. Create the server.  All strightforward `nodejs` stuff.

The difference lies in the function we pass into create server.  if the `req.method === 'POST` we pipe `req` (which is just a stream) to `through`.  In the function we pass into `through` we uppercase just like we did in Stream Adventure 4 - Transform.  The point is `req` and `res` are streams.

## 8. HTTP CLIENT

### Problem

Send an HTTP POST request to http://localhost:8099 and pipe process.stdin into
it. Pipe the response stream to process.stdout.

Here's an example of how to use the `request` module to send a GET request,
piping the result to stdout:

    var request = require('request');
    request('http://beep.boop:80/').pipe(process.stdout);

To make a POST request, just call `request.post()` instead of `request()`:

    var request = require('request');
    var r = request.post('http://beep.boop:80/');
    
The `r` object that you get back from `request.post()` is a readable+writable
stream so you can pipe a readable stream into it (`src.pipe(r)`) and you can
pipe it to a writable stream (`r.pipe(dst)`).

You can even chain both steps together: src.pipe(r).pipe(dst);

Hint: for your code, src will be process.stdin and dst will be process.stdout.

Make sure to `npm install request` in the directory where your solution file
lives.

### Solution
```javascript
var request = require('request');
var r = request.post('http://localhost:8099');
process.stdin.pipe(r).pipe(process.stdout);
```

Here we need the request module.  

We then take `process.stdin` and pipe it to our post request, then we pipe that to `process.stdout`.


