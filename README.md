# boot-cljs

[![Clojars Project][2]][3]

[Boot](https://github.com/boot-clj/boot) task to compile ClojureScript applications.

* Provides the `cljs` task–compiles ClojureScript to JavaScript.
* Provides a mechanism by which JS preamble, and Google Closure libs and externs
  files can be included in Maven jar dependencies or from the project source
  directories.

## Try It

In a terminal do:

```bash
mkdir src
echo -e '(ns foop)\n(.log js/console "hello world")' > src/foop.cljs
boot -s src -d adzerk/boot-cljs cljs
```

The compiled JavaScript will be written to `target/main.js`.

## Usage

Add `boot-cljs` to your `build.boot` dependencies and `require` the namespace:

```clj
(set-env! :dependencies '[[adzerk/boot-cljs "X.Y.Z" :scope "test"]])
(require '[adzerk.boot-cljs :refer :all])
```

You can see the options available on the command line:

```bash
boot cljs -h
```

or in the REPL:

```clj
boot.user=> (doc cljs)
```

### Compilation Levels

The ClojureScript compiler uses the Google Closure compiler to generate
optimized JavaScript when desired. There are [three different Closure
compilation levels][closure-levels]: `whitespace`, `simple`, and
`advanced`. You may specify the desired compilation level with the `-O`
option:

```bash
boot cljs -O advanced
```

The default level is `whitespace`. Additionally, the `none` level can be
specified to bypass the Closure compiler.

### Unified HTML

The HTML file will always need to have a `<script>` tag to load the compiled
JavaScript:

```html
<!-- compiling with optimizations != none -->
<script type='text/javascript' src='main.js'></script>
```

However, when the Closure compiler is bypassed (optimizations `none`)
two other `<script>` tags must be added to the HTML:

```html
<!-- compiling with optimizations == none -->
<script type='text/javascript' src='out/goog/base.js'></script>
<script type='text/javascript' src='main.js'></script>
<script type='text/javascript'>goog.require('my.namespace');</script>
```

The `-u` option may be used to automatically add the extra `<script>` tags when
compiling with `none` optimizations, so you don't need to have different HTML
for different compilation levels.

### Source Maps

[Source maps][src-maps] associate locations in the compiled JavaScript file with
the corresponding line and column in the ClojureScript source files. When source
maps are enabled (the `-s` option) the browser developer tools will refer to
locations in the ClojureScript source rather than the compiled JavaScript.

```bash
boot cljs -s
```

> You may need to enable source maps in your browser's developer tools settings.

### Incremental Builds

You can run boot such that it watches source files for changes and recompiles
the JavaScript file as necessary:

```bash
boot watch cljs
```

You can also get audible notifications whenever the project is rebuilt:

```bash
boot watch speak cljs
```

> **Note:** The `watch` and `speak` tasks are not part of `boot-cljs`–they're
> built-in tasks that come with boot.

### Browser REPL

See the [adzerk/boot-cljs-repl][boot-cljs-repl] boot task.

### Preamble, Externs, and Lib Files

The `cljs` task figures out what to do with these files by scanning for
resources on the classpath that have special filename extensions. They should
be under `hoplon/include/` in the source directory or jar file.

File extensions recognized by the `cljs` task:

* `.inc.js`: JavaScript preamble files–these are prepended to the compiled
  Javascript in dependency order (i.e. if jar B depends on jar A then entries
  from A will be added to the JavaScript file such that they'll be evaluated
  before entries from B).

* `.lib.js`: GClosure lib files (JavaScript source compatible with the Google
  Closure compiler).

* `.ext.js`: [GClosure externs files][closure-externs]–hints to the Closure
  compiler that prevent it from mangling external names under advanced
  optimizations.

## Examples

Create a new ClojureScript project, like so:

```
my-project
├── build.boot
├── html
│   └── index.html
└── src
    └── foop.cljs
```

and add the following contents to `build.boot`:

```clj
(set-env!
  :src-paths    #{"src"}
  :rsc-paths    #{"html"}
  :dependencies '[[adzerk/boot-cljs "0.0-X-Y" :scope "test"]])

(require '[adzerk.boot-cljs :refer :all])
```

Then in a terminal:

```bash
boot cljs -usO none
```

The compiled JavaScript file will be `target/main.js`.

### Preamble and Externs

Add preamble and extern files to the project, like so:

```
my-project
├── build.boot
├── html
│   └── index.html
└── src
    ├── foop.cljs
    └── hoplon
        └── include
            ├── barp.ext.js
            └── barp.inc.js
```

With the contents of `barp.inc.js` (a preamble file):

```javascript
(function() {
  window.Barp = {
    bazz: function(x) {
      return x + 1;
    }
  };
})();
```

and `barp.ext.js` (the externs file for `barp.inc.js`):

```javascript
var Barp = {};
Barp.bazz = function() {};
```

Then, in `foop.cljs` you may freely use `Barp`, like so:

```clj
(ns foop)

(.log js/console "Barp.bazz(1) ==" (.bazz js/Barp 1))
```

Compile with advanced optimizations and source maps:

```bash
boot cljs -usO advanced
```

You will see the preamble inserted at the top of `main.js`, and the references
to `Barp.bazz()` are not mangled by the Closure compiler. Whew!

### Further Reading
For an example project with a local web server, CLJS REPL, and live-reload, check out [boot-cljs-example](https://github.com/adzerk/boot-cljs-example)!


## License

Copyright © 2014 Adzerk

Distributed under the Eclipse Public License either version 1.0 or (at
your option) any later version.

[1]:                https://github.com/boot-clj/boot
[2]:                http://clojars.org/adzerk/boot-cljs/latest-version.svg?cache=6
[3]:                http://clojars.org/adzerk/boot-cljs
[cider]:            https://github.com/clojure-emacs/cider
[boot-cljs-repl]:   https://github.com/adzerk/boot-cljs-repl
[src-maps]:         https://developer.chrome.com/devtools/docs/javascript-debugging#source-maps
[closure-compiler]: https://developers.google.com/closure/compiler/
[closure-levels]:   https://developers.google.com/closure/compiler/docs/compilation_levels
[closure-externs]:  https://developers.google.com/closure/compiler/docs/api-tutorial3#externs
