# recursive-readdir-ext

Recursively list all files in a directory and its subdirectories. It does not list the directories themselves.

Because it uses fs.readdir, which calls [readdir](http://linux.die.net/man/3/readdir) under the hood on OS X and Linux, the order of files inside directories is [not guaranteed](http://stackoverflow.com/questions/8977441/does-readdir-guarantee-an-order).

The library is a based on [recursive-readdir](github.com/jergason/recursive-readdir.git) by Jamison Dance <jergason@gmail.com> (http://jamisondance.com/)

## Installation

    npm install recursive-readdir-ext

## Usage

```javascript
var recursive = require("recursive-readdir-ext");

recursive("some/path", function(err, files) {
  // `files` is an array of file paths
  console.log(files);
});
```

It can also take a list of files to ignore.

```javascript
var recursive = require("recursive-readdir");

// ignore files named "foo.cs" or files that end in ".html".
recursive("some/path", ["foo.cs", "*.html"], function(err, files) {
  console.log(files);
});
```

You can also pass functions which are called to determine whether or not to
ignore a file:

```javascript
var recursive = require("recursive-readdir");

function ignoreFunc(file, stats) {
  // `file` is the path to the file, and `stats` is an `fs.Stats`
  // object returned from `fs.lstat()`.
  return stats.isDirectory() && path.basename(file) == "test";
}

// Ignore files named "foo.cs" and descendants of directories named test
recursive("some/path", ["foo.cs", ignoreFunc], function(err, files) {
  console.log(files);
});
```

## Promises

You can omit the callback and return a promise instead.

```javascript
readdir("some/path").then(
  function(files) {
    console.log("files are", files);
  },
  function(error) {
    console.error("something exploded", error);
  }
);
```

The ignore strings support Glob syntax via
[minimatch](https://github.com/isaacs/minimatch).

## Custom file system

You can now pass in an options object in place of `ignores`, so that you can pass in a custom filesystem as an `fs` option. This is ideal for testing or when operating on an in-memory file system etc.

```js
import { fs, vol } from "memfs";

const fileStruct = {
  "./README.md": "# hello",
  "./src/index.js": "1 + 2;",
  "./node_modules/debug/index.js": "2 + 3;"
};
vol.fromJSON(fileStruct, "/app");

const successFn = files => console.log(files);
const errFn = err => console.error(err);
```

You can also pass debugging and logging options to monitor how the file structure is processed,
what files are ignore etc.

```js
// using promise API
recursive("/app", {
  ignores: ["README.md"],
  fs, // in-memory fs
  debug: true,
  log: (...msg) => console.log("readdir", ...msg)
}).then(successFn, errFn);
```
