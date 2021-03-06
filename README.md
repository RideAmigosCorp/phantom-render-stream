# phantom-render-stream

Render a webpage and get the image as a stream.

```
npm install phantom-render-stream
```

[![Build Status](https://travis-ci.org/e-conomic/phantom-render-stream.png)](https://travis-ci.org/e-conomic/phantom-render-stream)

It uses a pool of phantom processes so it doesn't need to spawn a new process for each website.
New requests are added to the pool member with the shortest queue length.

## Synopsis

This module depends on the [phantomjs](https://www.npmjs.org/package/phantomjs) module, which will install
`phantomjs` for you if you don't already have it.

``` js
var phantom = require('phantom-render-stream');
var fs = require('fs');

var render = phantom();
render('http://example.com/my-site').pipe(fs.createWriteStream('out.png'));
```

You can also pass some options:

``` js
var render = phantom({
  pool        : 5,           // Change the pool size. Defaults to 1
  timeout     : 1000,        // Set a render timeout. No default
  tmp         : '/tmp',      // Set the tmp where tmp data is stored when communicating with the phantom process.
  format      : 'jpeg',      // The default output format. Defaults to png
  width       : 1280,        // Changes the width size. Defaults to 1280
  height      : 800,         // Changes the height size. Defaults to 960
  paperFormat : 'A4',        // Defaults to A4. Also supported: 'A3', 'A4', 'A5', 'Legal', 'Letter', 'Tabloid'.
  orientation : 'portriat',  // Defaults to portrait. 'landscape' is also valid
  margin      : '0cm',       // Defaults to 0cm. Supported dimension units are: 'mm', 'cm', 'in', 'px'. No unit means 'px'.
  userAgent   : '',          // No default.
  crop        : false,       // Defaults to false. Set to true or {top:5, left:5} to add margin
  printMedia  : false,       // Defaults to false. Force the use of a print stylesheet.
  expects     : 'something', // No default. Do not render until window.renderable is set to 'something'
  phantomFlags: ['--ignore-ssl-errors=true'] // Defaults to []. Command line flags passed to phantomjs 
});
```

Or override the options for each render stream

``` js
render(myUrl, {format:'jpeg', width: 1280, height: 960}).pipe(...)
```

## Supported output formats

We support the output formats that [Phantom's render method](http://phantomjs.org/api/webpage/method/render.html)
supports. At the time of this writing these are:

 * png
 * gif
 * jpg
 * pdf

## Example

Since the interface is just a stream you can pipe the web site anywhere!
Try installing [picture-tube](https://github.com/substack/picture-tube) and run the following example

``` js
var phantom = require('phantom-render-stream');
var pictureTube = require('picture-tube');
var render = phantom();

render('http://google.com')
  .pipe(pictureTube())
  .pipe(process.stdout);
```

## Deferred render

If you need your page to do something before phantom renders it you just need to immediately set
`window.renderable` to false. If that is set when the page is opened the module will wait for 
`window.renderable` to be set to true and when this happens the render will occur.

Here is an example to illustrate it better.

``` html
<!DOCTYPE HTML>
<html lang="en">
<head>
  ...
  <script type="text/javascript">window.renderable = false</script>
  <meta charset="UTF-8">
  <title></title>
</head>
<body>

</body>
...
<script type="text/javascript">
  doSomeAjaxLoading(function() {
    doSomeRendering();
    window.renderable = true;
  })
</script>
</html>
```

## License

[MIT](http://opensource.org/licenses/MIT)
