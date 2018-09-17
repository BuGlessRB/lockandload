<h1>lockandload AMD-loader</h1>

Lockandload is a minimalist AMD-loader-compatible boilerplate to kickstart
your website.  It includes special support for single-page-apps.

Gzipped, the essential script content amounts to roughly 859 bytes of code.
Without compression it blows up to about 1520 bytes.  Further minifying
this code does not result in any significant gains (773 vs 859 bytes), it
would just hinder readability.

## Features

- Less filling: 859 bytes of gzipped script content.
- Handminified to retain readable and maintainable code.
- It's so small, it can and should be inlined on your HTML page
  (which is also one of the reasons to handminify it only).
- Because it is inlined, it is faster than all other loaders.
- Fully asynchronous script loader: AMD-loader compatible.
- Supports anonymous define() calls.
- Supports local require() calls (with one and two arguments, RequireJS-style).
- Supports require.undef() for hot-reloading scripts.
- Supports require.load() to load scripts on demand.
- Supports implicit and explicit ['require', 'exports', 'module'] dependencies.
- Explicit circular dependencies will silently hang in unresolved state
  (or put differently: do not do that), use 'exports' or 'require' to resolve
  those instead.
- No extra diagnostic code to minimise code weight and optimise loading speed.
- Does not support require.toUrl() nor simplified CommonJS wrapping.
- Fully event driven, no polling timers.
- Standard supported dependencies: `require`, `exports`, `module`
  and `domready`.
- Both high and low priority asynchronous loading of Javascript and CSS files.
- Leverages native browser speed for high priority loading (by getting out
  of the way).
- Legacy support for $(...) jquery riddled synchronous code.
- Legacy support for loading synchronous Javascript.
- Single-page-app support using $$(...) page refresh callbacks.
- Supports IE10 and up and all other webbrowsers.
- No config file, means: no syntax to learn, no config file parser code.
- No module system: all needed functionality is included already because
  it was/is so small, that writing a module system would take more code
  than the source of all the added functionality.
- Integrated hooks for Google Tag Manager (GTM) support.

## Requirements

It runs inside any webbrowser environment (starting at IE10 and up).

## Usage

### Using npm

Running `npm install lockandload` in the webroot of your site,
should create the following file and directory structure:
- `node_modules`
  - `lockandload`
    - `lockandload_master.inc`: Placed right after the charset definition
       on the page.
    - `lockandload_headready.inc`: The start of the headready-script that
       lives at the end of the `<head>`.
    - `lockandload_trailer.inc`: The end of the headready-script that lives
       at the end of the `<head>`.
    - `index.php`: PHP boilerplate.
    - `index_inlined.html`: High performance HTML only boilerplate.
    - `lockandload_master.js`: For lazy `<script>` loading.
    - `lockandload_headready.js`: Boilerplate for lazy `<script>` loading.
    - `index_extern.html`: Lazy HTML only boilerplate.
    - `main.js`: Example SPA (Single Page Application).

### Using PHP

Copy the boilerplate `node_modules/lockandload/index.php` file to your
webroot; then customise the copied file to taste.

### Using other serverside scripting languages

Look at the PHP boilerplate `node_modules/lockandload/index.php`, and
translate this to your own scripting language.

### Without serverside scripting (lazy)

Copy the `node_modules/lockandload/index_extern.html` boilerplate
file to your webroot; then customise the copied file to taste.
Copy the `node_modules/lockandload/lockandload_headready.js` file
to your javascript directory and customise it taste.  Do not forget
to change the path of the `script` directive in `index_extern.html`
to point to the new location of the headready script.

### Without serverside scripting (high performance)

Copy the `node_modules/lockandload/index_inlined.html` boilerplate
file to your webroot; then customise the copied file to taste.
The `index_inlined.html` contains two `<script>` sections.  The first section
should not be preceded by any other `<script>` tags and should be left
verbatim.

The second section should be placed at or close to the end of the `<head>`,
and should not precede any direct `<link type="stylesheet">` tags.
Inside this second section there is a clearly marked section that is
your configuration area.

The basic structure of a page should be:
- html
  - head
    - Charset declaration.
    - Inline `lockandload` master script.
    - High priority async external scripts.
    - Viewport declaration.
    - High priority CSS scripts.
    - `<title>`.
    - All other tags that should go in the `<head>`.
    - Inline `lockandload` headready-script.
      - CSS scripts fullfilling a custom applied-style dependency.
      - Low priority CSS scripts.
      - Medium priority async Javascript scripts.
      - Low priority async Javascript scripts.
      - Low priority synchronous Javascript scripts.
  - body
    - All other inline scripts (if you must).

The `index_inlined.html` file is a production-stripped version
of `annotated.html`.  Look at `annotated.html` to understand the code and
read additional inline documentation.  These `index.*`, `lockandload_*.inc`
and `lockandload_*.js`
files are not present in the git source repository, they can only be found in
the npm repository (or after running `npm run prepublish`).

## API

### Globally
- `define(id?, dependencies?, factory)`<br />
   The standard [AMD global
    entrypoint](https://github.com/amdjs/amdjs-api/blob/master/AMD.md).
   To figure out module ids of all
   the modules that you are trying to load, uncomment some debugging code
   in the primary load script and inspect your console-pane in the browser.

   If you are in need of the common global `require(dependencies, factory)`,
   insert the following code into the loader (preferably in the custom-code
   section of the secondary headready-script):
```javascript
   function require(d, f) { define(1, d, f); }
```

### Locally
In the secondary `lockandload` headready-script; all url arguments
are used verbatim in `<link href="url">` or `<script src="url">` tags:
- `css(url, id?)`<br />
   Loads low priority ordered css files asynchronously;
   after the stylesheet has been applied, it fulfills the optional
   `id` dependency.
- `js(url, "async"?)`<br />
   Loads Javascript file, if the second optional argument `"async"` is
   provided, the load will be asynchronous.
- `jsa(alias, path)`<br />
   Define aliases for javacsript file paths to be referenced through
   `require.load(alias)` to load the file on demand.

### Dealing with jQuery
In order to support legacy code that uses inline `$(function(){...})` scattered
throughout pages, this loader allows you to use that construct even before
the jQuery library has been loaded,
and thus enables you to load jQuery in an asynchronous and non-blocking fashion.

The standard headready-script contains a dependency on `domready` and
`jquery`  which finally runs `domready(1)` which will run all the
registered delayed functions the first time.

### SPA (Single Page App) support
To ease SPA development, the loader defines a
`$$(function(jquery_document){...})`  function which registers functions
for execution on every SPA-controlled page refresh.  The registered functions
receive a convenience argument `$(document)` when executed.

To run the registered functions, one needs to make a call to the entrypoint
of the AMD-dependency on `domready` without parameters or with exactly
one parameter; if not provided, this single parameter will default to
`$(document)` (the jquery object/scope referring to the whole document).
Convention states that if provided the argument should normally be the jquery
object referring to the element tree that contains the changes.
Ultimately you decide what your `$$(function(argument){...})` scheduled
scripts will use the argument for.  All `domready()`
calls before `domready(1)` has been run will silently be ignored.

E.g. in your application, you could use code like this:

```javascript
!function(){
  // Preamble
  define("main", ["domready"], function (domready) {
    // Your main application
    function refreshpage() {
      // The function that gets called on virtual page refreshes
      var newdiv = $("#contentdiv");
      newdiv.html("your glorious new page content");
      domready(newdiv);   // This will call all registered $$(...) functions
                          // with the newdiv as argument to potentially
                          // limit the scope of the changes
    }
  });
}();
```

## References

- To install it use the
  [lockandload AMD-loader](https://www.npmjs.com/package/lockandload)
  npm repository.
- [lockandload AMD-loader](https://github.com/BuGlessRB/lockandload)
  source repository.
- Sample site that uses lockandload: [Remixml](https://remixml.org/).
- [Asynchronous Module Definition
   (AMD)](https://github.com/amdjs/amdjs-api/blob/master/AMD.md).

Other loaders:
- [eeMD](https://github.com/MaxMotovilov/eeMD).
- [Curiosity-Driven](https://curiosity-driven.org/minimal-loader).
- [RequireJS](https://requirejs.org/).
- [curl](https://github.com/cujojs/curl).
- [Dojo Toolkit](https://dojotoolkit.org/).
- [lsjs](https://github.com/zazl/lsjs).
- [amdlite](https://github.com/abadc0de/amdlite).
- [tinyamd](https://github.com/briancray/tinyamd).
- [tAMD](https://github.com/jivesoftware/tAMD).
- [almond](https://github.com/requirejs/almond).
- [bdLoad](http://bdframework.org/bdLoad/).
- [yabble](https://github.com/jbrantly/yabble).
- [PINF](https://github.com/pinf/loader-js).
- [nodules](https://github.com/kriszyp/nodules).
