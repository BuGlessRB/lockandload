<h1>lockandload AMD-loader</h1>

Lockandload is a minimalist AMD-loader-compatible boilerplate to kickstart
your website.  It includes special support for single-page-apps.

Gzipped, the essential script content amounts to roughly 859 bytes of code.
Without compression it blows up to about 1520 bytes.  Further minifying
this code does not result in any significant gains (773 vs 859 bytes), it
would just hinder readability.

## Why

After reviewing the javascript-loader landscape, looking for a loader
that satisfies the following criteria:
- Small enough to be inlined (to avoid paying an extra request-tax to
  get the loader onboard).
- Inlined config file (get rid of a request).
- Config file in pure javascript (no parsing overhead).
- Let the browser do the actual loading (browsers are smart these days)
  of files.
- Connect all asynchronously loaded modules together.
- Support for single page apps that include legacy code that
  uses sprinkled `$(function(){...})` constructs, yet I insist on loading
  jQuery late and asynchronously to speed things up.

I came to the conclusion that (apparently?) none of the existing loaders
fit the bill.  So I wrote `lockandload` back in 2014 and used it in
various internal projects, but decided to open-source
it now in case others find it useful.  As always, when open-sourcing something,
actually writing documentation takes most of the time.  The documentation
can always be improved.  Suggestions are welcome!

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
- Legacy support for $(...) jQuery riddled synchronous code.
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
    - `lockandload_master_debug.inc`: Alternate debugging version of
       `lockandload_master.inc`.
    - `index.php`: PHP boilerplate.
    - `index_inlined.html`: High performance HTML only boilerplate.
    - `lockandload_master.js`: For lazy `<script>` loading.
    - `lockandload_master_debug.js`: Alternate debugging version of
       `lockandload_master.js`.
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

### Module ids

Module ids are short strings that uniquely identify a module.
In `lockandload` these ids typically do not contain parts of a path.
If a module id is derived from the javascript filename that is being
loaded, it will only refer to the final path component without
`.js` or version extension (i.e. anything after the last `/` and
before the first `.`).

### Globally
- `define(id?, dependencies?, factory)`<br />
   The standard [AMD global
    entrypoint](https://github.com/amdjs/amdjs-api/blob/master/AMD.md).
   - `id` declares the module id we are defining.  If omitted, we derive
     a module id from the name of the javascript file we are loading.
   - `dependencies` is an array of strings of module ids this module depends on.
     If the parameter is missing, a default dependency list of
     `["require", "exports", "module"]` is supplied.
   - `factory` is the callback function that gets called as soon as
     all dependencies have been loaded.  The factory function gets
     references to all the exported symbols from its dependencies, and
     should subsequently return its own symbols it wants to export to
     other modules.  The factory can be a function or a static object.

- `require(dependencies, callback)`<br />
   Allows you to load dependencies (an array of strings of module ids)
   asynchronously, the callback is called
   as soon as all dependencies have loaded.  Parameters to the callback
   are references to the dependencies just as in the factory function
   in `define`.

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
   Define aliases for javascript file paths to be referenced through
   `require.load(alias)` to load the file on demand.

### Standard require module

If you use `"require"` in your dependencylist, you get a reference to the
internal require-module.  It supports the following functionality:
- `require(id)`<br />
   Returns a reference to the exports of module `id`.  Beware:
   this will not trigger loading that module, if the module is not loaded
   because it was already in your dependencylist, or if the module is
   not being requested by other modules, this will return zero.
   As such, it can be used to find out if a certain module is being loaded
   at all.
- `require.undef(id)`<br />
   Clears the cached module `id` exported symbols list.  This allows the module
   to be hot-reloaded by a subsequent `require.load()`.  Beware that existing
   references to the old module are not overwritten.  Any modules using
   `require(id)` before the `require.load()` will return zero, any modules
   using `require(id)` after the `require.load()` will return a reference
   to the exports of the reloaded module.
- `require.load(file)`<br />
   Asynchronously loads the referenced javascript file.  To centralise
   file-location management, it is advisable to use `jsa()` calls in the
   headready section to declare aliases for javascript files which can
   be used instead of actual file paths in the `require.load()` calls.

### Dealing with jQuery
In order to support legacy code that uses inline `$(function(){...})` scattered
throughout pages, this loader allows you to use that construct even before
the jQuery library has been loaded,
and thus enables you to load jQuery in an asynchronous and non-blocking fashion.

N.B. Use the `jQuery` dependency instead of plain `jquery` to avoid a race
for an undefined `window.$` object.

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
- [CrocoDillon](https://github.com/CrocoDillon/define).
- [tAMD](https://github.com/jivesoftware/tAMD).
- [almond](https://github.com/requirejs/almond).
- [bdLoad](http://bdframework.org/bdLoad/).
- [yabble](https://github.com/jbrantly/yabble).
- [PINF](https://github.com/pinf/loader-js).
- [nodules](https://github.com/kriszyp/nodules).
- [Liferay](https://github.com/liferay/liferay-amd-loader).
- [lodJS](https://github.com/yanhaijing/lodjs).
