# api documentation for  [hooker (v0.2.3)](http://github.com/cowboy/javascript-hooker)  [![npm package](https://img.shields.io/npm/v/npmdoc-hooker.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-hooker) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-hooker.svg)](https://travis-ci.org/npmdoc/node-npmdoc-hooker)
#### Monkey-patch (hook) functions for debugging and stuff.

[![NPM](https://nodei.co/npm/hooker.png?downloads=true)](https://www.npmjs.com/package/hooker)

[![apidoc](https://npmdoc.github.io/node-npmdoc-hooker/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-hooker_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-hooker/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-hooker/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-hooker/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "\"Cowboy\" Ben Alman",
        "url": "http://benalman.com/"
    },
    "bugs": {
        "url": "https://github.com/cowboy/javascript-hooker/issues"
    },
    "dependencies": {},
    "description": "Monkey-patch (hook) functions for debugging and stuff.",
    "devDependencies": {
        "grunt": "~0.2.1"
    },
    "directories": {},
    "dist": {
        "shasum": "b834f723cc4a242aa65963459df6d984c5d3d959",
        "tarball": "https://registry.npmjs.org/hooker/-/hooker-0.2.3.tgz"
    },
    "engines": {
        "node": "*"
    },
    "homepage": "http://github.com/cowboy/javascript-hooker",
    "keywords": [
        "patch",
        "hook",
        "function",
        "debug",
        "aop"
    ],
    "licenses": [
        {
            "type": "MIT",
            "url": "https://github.com/cowboy/javascript-hooker/blob/master/LICENSE-MIT"
        }
    ],
    "main": "lib/hooker",
    "maintainers": [
        {
            "name": "cowboy",
            "email": "cowboy@rj3.net"
        }
    ],
    "name": "hooker",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git://github.com/cowboy/javascript-hooker.git"
    },
    "scripts": {
        "test": "grunt test"
    },
    "version": "0.2.3"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module hooker](#apidoc.module.hooker)
1.  [function <span class="apidocSignatureSpan">hooker.</span>filter (context, args)](#apidoc.element.hooker.filter)
1.  [function <span class="apidocSignatureSpan">hooker.</span>hook (obj, props, options)](#apidoc.element.hooker.hook)
1.  [function <span class="apidocSignatureSpan">hooker.</span>orig (obj, prop)](#apidoc.element.hooker.orig)
1.  [function <span class="apidocSignatureSpan">hooker.</span>override (value)](#apidoc.element.hooker.override)
1.  [function <span class="apidocSignatureSpan">hooker.</span>preempt (value)](#apidoc.element.hooker.preempt)
1.  [function <span class="apidocSignatureSpan">hooker.</span>unhook (obj, props)](#apidoc.element.hooker.unhook)



# <a name="apidoc.module.hooker"></a>[module hooker](#apidoc.module.hooker)

#### <a name="apidoc.element.hooker.filter"></a>[function <span class="apidocSignatureSpan">hooker.</span>filter (context, args)](#apidoc.element.hooker.filter)
- description and source-code
```javascript
filter = function (context, args) {
  return new HookerFilter(context, args);
}
```
- example usage
```shell
...
#### Signature:
'hooker.preempt(value)'

### hooker.filter
When a pre-hook returns the result of this function, the context and
arguments passed will be applied into the original function.
#### Signature:
'hooker.filter(context, arguments)'


## Examples
See the unit tests for more examples.

'''javascript
var hooker = require('hooker');
...
```

#### <a name="apidoc.element.hooker.hook"></a>[function <span class="apidocSignatureSpan">hooker.</span>hook (obj, props, options)](#apidoc.element.hooker.hook)
- description and source-code
```javascript
hook = function (obj, props, options) {
  // If the props argument was omitted, shuffle the arguments.
  if (options == null) {
    options = props;
    props = null;
  }
  // If just a function is passed instead of an options hash, use that as a
  // pre-hook function.
  if (typeof options === "function") {
    options = {pre: options};
  }

  // Hook the specified method of the object.
  return forMethods(obj, props, function(obj, prop) {
    // The original (current) method.
    var orig = obj[prop];
    // The new hooked function.
    function hooked() {
      var result, origResult, tmp;

      // Get an array of arguments.
      var args = slice.call(arguments);

      // If passName option is specified, prepend prop to the args array,
      // passing it as the first argument to any specified hook functions.
      if (options.passName) {
        args.unshift(prop);
      }

      // If a pre-hook function was specified, invoke it in the current
      // context with the passed-in arguments, and store its result.
      if (options.pre) {
        result = options.pre.apply(this, args);
      }

      if (result instanceof HookerFilter) {
        // If the pre-hook returned hooker.filter(context, args), invoke the
        // original function with that context and arguments, and store its
        // result.
        origResult = result = orig.apply(result.context, result.args);
      } else if (result instanceof HookerPreempt) {
        // If the pre-hook returned hooker.preempt(value) just use the passed
        // value and don't execute the original function.
        origResult = result = result.value;
      } else {
        // Invoke the original function in the current context with the
        // passed-in arguments, and store its result.
        origResult = orig.apply(this, arguments);
        // If the pre-hook returned hooker.override(value), use the passed
        // value, otherwise use the original function's result.
        result = result instanceof HookerOverride ? result.value : origResult;
      }

      if (options.post) {
        // If a post-hook function was specified, invoke it in the current
        // context, passing in the result of the original function as the
        // first argument, followed by any passed-in arguments.
        tmp = options.post.apply(this, [origResult].concat(args));
        if (tmp instanceof HookerOverride) {
          // If the post-hook returned hooker.override(value), use the passed
          // value, otherwise use the previously computed result.
          result = tmp.value;
        }
      }

      // Unhook if the "once" option was specified.
      if (options.once) {
        exports.unhook(obj, prop);
      }

      // Return the result!
      return result;
    }
    // Re-define the method.
    obj[prop] = hooked;
    // Fail if the function couldn't be hooked.
    if (obj[prop] !== hooked) { return false; }
    // Store a reference to the original method as a property on the new one.
    obj[prop]._orig = orig;
  });
}
```
- example usage
```shell
...

This code should work just fine in Node.js:

First, install the module with: 'npm install hooker'

'''javascript
var hooker = require('hooker');
hooker.hook(Math, "max", function() {
  console.log(arguments.length + " arguments passed");
});
Math.max(5, 6, 7) // logs: "3 arguments passed", returns 7
'''

Or in the browser:
...
```

#### <a name="apidoc.element.hooker.orig"></a>[function <span class="apidocSignatureSpan">hooker.</span>orig (obj, prop)](#apidoc.element.hooker.orig)
- description and source-code
```javascript
orig = function (obj, prop) {
  return obj[prop]._orig;
}
```
- example usage
```shell
...
The optional 'props' argument can be a method name, array of method names or null. If null (or omitted), all methods of 'object'
will be unhooked.
#### Returns:
An array of unhooked method names.

### hooker.orig
Get a reference to the original method from a hooked function.
#### Signature:
'hooker.orig(object, props)'

### hooker.override
When a pre- or post-hook returns the result of this function, the value
passed will be used in place of the original function's return value. Any
post-hook override value will take precedence over a pre-hook override value.
#### Signature:
'hooker.override(value)'
...
```

#### <a name="apidoc.element.hooker.override"></a>[function <span class="apidocSignatureSpan">hooker.</span>override (value)](#apidoc.element.hooker.override)
- description and source-code
```javascript
override = function (value) {
  return new HookerOverride(value);
}
```
- example usage
```shell
...
'hooker.orig(object, props)'

### hooker.override
When a pre- or post-hook returns the result of this function, the value
passed will be used in place of the original function's return value. Any
post-hook override value will take precedence over a pre-hook override value.
#### Signature:
'hooker.override(value)'

### hooker.preempt
When a pre-hook returns the result of this function, the value passed will
be used in place of the original function's return value, and the original
function will NOT be executed.
#### Signature:
'hooker.preempt(value)'
...
```

#### <a name="apidoc.element.hooker.preempt"></a>[function <span class="apidocSignatureSpan">hooker.</span>preempt (value)](#apidoc.element.hooker.preempt)
- description and source-code
```javascript
preempt = function (value) {
  return new HookerPreempt(value);
}
```
- example usage
```shell
...
'hooker.override(value)'

### hooker.preempt
When a pre-hook returns the result of this function, the value passed will
be used in place of the original function's return value, and the original
function will NOT be executed.
#### Signature:
'hooker.preempt(value)'

### hooker.filter
When a pre-hook returns the result of this function, the context and
arguments passed will be applied into the original function.
#### Signature:
'hooker.filter(context, arguments)'
...
```

#### <a name="apidoc.element.hooker.unhook"></a>[function <span class="apidocSignatureSpan">hooker.</span>unhook (obj, props)](#apidoc.element.hooker.unhook)
- description and source-code
```javascript
unhook = function (obj, props) {
  return forMethods(obj, props, function(obj, prop) {
    // Get a reference to the original method, if it exists.
    var orig = exports.orig(obj, prop);
    // If there's no original method, it can't be unhooked, so fail.
    if (!orig) { return false; }
    // Unhook the method.
    obj[prop] = orig;
  });
}
```
- example usage
```shell
...

#### Returns:
An array of hooked method names.

### hooker.unhook
Un-monkey-patch (unhook) one or more methods of an object.
#### Signature:
'hooker.unhook(object [, props ])'
#### 'props'
The optional 'props' argument can be a method name, array of method names or null. If null (or omitted), all methods of 'object'
will be unhooked.
#### Returns:
An array of unhooked method names.

### hooker.orig
Get a reference to the original method from a hooked function.
...
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
