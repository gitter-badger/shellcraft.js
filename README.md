
# shellcraft.js

IT'S WORK IN PROGRESS, DON'T USE IT IN PROD.

Simple shell for Node.js based on commander and inquirer.

## Documentation

### Installation

```shell
$ npm install shellcraft
```

### Hello, World

```javascript
var shellcraft = require ('shellcraft');

var options = {
  version: '0.1.0'
};

shellcraft.begin ({}, function (msg) {
  if (msg) {
    console.log (msg);
  }
});
```

### API

There are only two public API in order to play with shellcraft.

---
#### shellcraft.begin (options, callback)

✤ options

Some options are available through the `option` argument.

```javascript
options = {
  version: '0.1.0',
  prompt: '>'
}
```

The `version` is used by Commander with the `-V, --version` parameter.
The default prompt `>` can be changed by something else. But note that the
prompt will always begins by `? `, like for example:

```shell
? myprompt>
```

it's because Inquirer has already its own prompt and this one can not be changed
easily.

✤ callback (msg)

The callback is called when the shell or the CLI is terminated. Note that
currently the `msg` argument is not consistent between the CLI and the shell.
This behavior will change in the future.

##### Example

```javascript
shellcraft.begin ({
  version: '0.0.1',
  prompt: 'orc>'
}, function (msg) {
  if (msg) {
    console.log (msg);
  }
});
```

shell mode
```
$ node myShell.js
? orc> _
? orc> help
 exit     exit the shell
 help     list of commands
? orc> exit
good bye
$ _
```

CLI mode
```
$ node myShell.js -h

  Usage: myShell [options]

  Options:

    -h, --help         output usage information
    -V, --version      output the version number

$ _
```

---
#### shellcraft.registerExtension (shellExt, callback);

There are two builtin commands `help` and `exit`. For more commands you must
register one or more extensions. An extension must be "requirable" and must
export an array of command definitions.

✤ shellExt

The path on the `.js` file where the definitions are exported. The array of
commands must be described like this:

```javascript
[{
  name    : 'foo',                        /* command's name without space     */
  desc    : 'foo description',            /* command's description (for help) */
  options : {
    wizard : false,                        /* when it's need Inquirer          */
    params : 'argName'                     /* only one argument allows         */
  },
  handler : function (callback, args) {
    /*
     * callback (wizard, function (answers) {})
     *   Is called in order to return to the prompt (or end if CLI). The wizard
     *   argument must be used only in order to process an Inquirer definition
     *   in the shell (or the CLI). Otherwise you must call the callback without
     *   arguments.
     *   The Inquirer answers are retrieved with the second argument.
     *
     * args
     *   Are the arguments provided with the command. Note that only the first
     *   argument is correctly handled.
     */
  }
}]
```

Your shell extension must provide two methods, `register` and `unregister`
functions.

✤ callback ()

The callback is called as soon as the extension is registered.

##### Example

`myShellExtension.js`

```javascript
'use strict';

exports.register = function (callback) {
  var commands = [{
    name    : 'hello',
    desc    : 'print Hello, John',
    options : {
      wizard : false,
      params : 'who'
    },
    handler : function (callback, args) {
      console.log ('Hello, ' + args[0]);
      callback ();
    }
  }, {
    name    : 'wizard',
    desc    : 'begins a wizard',
    options : {
      wizard : true
    },
    handler : function (callback, args) {
      var wizard = [{
        /* Inquirer definition... */
        type: 'input',
        name: 'zog',
        message: 'tell zog'
      }];

      callback (wizard, function (answers) {
        /* stuff on answers */
        if (answers.zog === 'zog') {
          console.log ('zog zog');
        } else {
          console.log ('lokthar?');
        }

        /*
         * You can return false if you must provide several wizard with only
         * one call to this command handler.
         * You can call callback () without argument in order to return to the
         * prompt instead of returning true.
         */
        return true;
      });
    }
  }];

  callback (commands);
};

exports.unregister = function () {
  /* internal stuff */
};
```

`myShell.js`

```javascript
'use strict';

var path       = require ('path');
var shellcraft = require ('../');

var options = {
  version: '0.0.1',
  prompt: 'orc>'
};
var shellExt = path.join (__dirname, 'myShellExtension.js');

shellcraft.registerExtension (shellExt, function () {
  shellcraft.begin (options, function (msg) {
    if (msg) {
      console.log (msg);
    }
  });
});
```

shell mode
```
$ node myShell.js
? orc> _
? orc> help
 exit      exit the shell
 help      list of commands
 hello     print Hello, John
 wizard    begins a wizard
? orc> hello Tux
Hello, Tux
? orc> exit
good bye
$ _
```

CLI mode
```
$ node myShell.js -h

  Usage: myShell [options]

  Options:

    -h, --help         output usage information
    -V, --version      output the version number

    hello <who>        print Hello, John
    wizard             begins a wizard

$ _
$ node myShell.js hello Alice
Hello, Alice
$ _
```
