# MuTeX - a fast, streaming Node.js Mustache engine *optimized for TeX*

## Differences from Mustache API

- Delimiter defaults changed from `{{ }}` to `<! !>` to avoid awkwardness
- Triple Mu tags (`{{{ }}}`) disabled as there is no need to convert to HTML
- Normal Mu tags `<! !>` escape text for LaTeX compatibility rather than HTML
- An `\input{nameoftemplate}` acts the same way that a partial (`{{> nameoftemplate }}`) does
- The `#` (if / iterate) command replaced with `*` to avoid problems
- The `^` (not) command replaced with `~` to avoid problems

#### Why don't you just use `<% %>` for the delimiters?
Because I want to do something like this:
``` tex
\renewcommand{\title}{<% title %>}
\renewcommand{\today}{<% today %>}
\renewcommand{\currentpart}{Revision <% revision %>}
```
Which (if you look at the gray regions) causes a loss of some of the important `}` signs that we need if we want to compile the documents in some *normal* way.  It's easier to just avoid the nasty syntax and use something like this:
``` tex
\renewcommand{\title}{<! title !>}
\renewcommand{\today}{<! today !>}
\renewcommand{\currentpart}{Revision <! revision !>}
```
Renders just fine no matter whether you're using TeXstudio or emacs to write your TeX, or Node to publish.

## Usage

There are a few ways to use mu 0.5. Here is the simplest:
```javascript
var mu = require('mutex'); // notice the "2" which matches the npm repo, sorry..

mu.root = __dirname + '/templates'
mu.compileAndRender('index.html', {name: "john"})
  .on('data', function (data) {
    console.log(data.toString());
  });
```
Here is an example mixing it with the http module:
```javascript
var http = require('http')
  , util = require('util')
  , mu   = require('mu2');

mu.root = __dirname + '/templates';

  http.createServer(function (req, res) {

  var stream = mu.compileAndRender('index.html', {name: "john"});
  util.pump(stream, res);

}).listen(8000);
```
Taking that last example here is a little trick to always compile the templates
in development mode (so the changes are immediately reflected).
```javascript
var http = require('http')
  , util = require('util')
  , mu   = require('mutex');

mu.root = __dirname + '/templates';

http.createServer(function (req, res) {

  if (process.env.NODE_ENV == 'DEVELOPMENT') {
    mu.clearCache();
  }

  var stream = mu.compileAndRender('index.html', {name: "john"});
  util.pump(stream, res);

}).listen(8000);
```
## API

    mu.root

      A path to lookup templates from. Defaults to the working directory.


    mu.compileAndRender(String templateName, Object view)

      Returns: Stream

      The first time this function is called with a specific template name, the
      template will be compiled and then rendered to the stream. Subsequent
      calls with the same template name will use a cached version of the compiled
      template to improve performance (a lot).


    mu.compile(filename, callback)

      Returns nil
      Callback (Error err, Any CompiledTemplate)

      This function is used to compile a template. Usually you will not use it
      directly but when doing wierd things, this might work for you. Does not
      use the internal cache when called multiple times, though it does add the
      compiled form to the cache.


    mu.compileText(String name, String template, Function callback)

      Returns nil
      Callback (err, CompiledTemplate)

      Similar to mu.compile except it taks in a name and the actual string of the
      template. Does not do disk io. Does not auto-compile partials either.


    mu.render(Mixed filenameOrCompiledTemplate, Object view)

      Returns Stream

      The brother of mu.compile. This function takes either a name of a template
      previously compiled (in the cache) or the result of the mu.compile step.

      This function is responsible for transforming the compiled template into the
      proper output give the input view data.


    mu.renderText(String template, Object view, Object partials)

      Returns Stream

      Like render, except takes a template as a string and an object for the partials.
      This is not a very performant way to use mu, so only use this for dev/testing.


    mu.clearCache(String templateNameOrNull)

      Clears the cache for a specific template. If the name is omitted, clears all cache.



