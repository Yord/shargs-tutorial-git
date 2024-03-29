![tutorial teaser][teaser]

shargs-tutorial-git is a tutorial for getting started with [shargs][shargs] 🦈.

See the [`shargs` github repository][shargs] for more details!

[![node version][shield-node]][node]
[![license][shield-license]][license]
[![PRs Welcome][shield-prs]][contribute]

## Code

The code we will write in this tutorial is available as the `git` script in this repository.
To run it, first setup the dependencies:

```bash
$ git clone https://github.com/Yord/shargs-tutorial-git.git
$ cd shargs-tutorial-git
$ npm i
$ chmod +x ./git
```

Then execute the script:

```bash
$ ./git --help
```

## Tutorial

In this tutorial, we will build a command-line interface that resembles `git`.
I assume, you are already familiar with `git`, but even if you are not, you should be able to follow.

Let us start with installing some shargs packages:

```bash
$ npm install --save --save-exact shargs@0.26.0
```

Note that we install a fixed version.
This tutorial should work with this exact version, and might work with a newer version.
However, this tutorial may not work if you use a different version.

### Parser

We start with a minimal shargs parser:

```js
#!/usr/bin/env node

const {parserSync} = require('shargs')

const parser = parserSync()
const parse  = parser()

const argv   = process.argv.slice(2)
const res    = parse(argv)

console.log(JSON.stringify(res, null, 2))
```

We choose to write a synchronous parser by importing `parserSync`.
Next, we initialize a new `parser`.
Calling the `parser` returns a `parse` function.

The `parse` function is then used to process `argv` argument values.
Note, that we skip the first two parameters of `process.argv`.
Those are always `node` and the file name.
Finally, we `log` the result to the `console`.

Let us now call this minimal shargs parser to see what it prints out.

<details>
<summary>
<code>$ ./git</code>
</summary>

<br />

```json
{
  "errs": [
    {
      "code": "CommandExpected",
      "msg": "Expected a command with a string 'key' field and an 'opts' array.",
      "info": {"opt": {}}
    }
  ],
  "args": {
    "_": []
  }
}
```

`parser` reports a `CommandExpected` error.
The shargs documentation has a table of error codes,
where we can look up [`CommandExpected`](https://github.com/Yord/shargs/tree/0.24.4#CommandExpected).
We find out, that the error is thrown in the `toOpts` stage.

The problem is, that we did not provide an `opt` parameter to `parser`.
Note though, that the parser did work anyway, by reporting an error, helping us along the way.

</details>

### Command

Our first `parser` did not parse anything, but instead reported an error.
We can make it work by adding a `command`:

```js
#!/usr/bin/env node

const {parserSync} = require('shargs')
const {command} = require('shargs/opts')

const git    = command('git', [])

const parser = parserSync()
const parse  = parser(git)

const argv   = process.argv.slice(2)
const res    = parse(argv)

console.log(JSON.stringify(res, null, 2))
```

We added a `git` `command` that has no options, yet.
`git` is then passed to `parser`.
Let us have a look at what changed.

<details>
<summary>
<code>./git --help</code>
</summary>

<br />

```json
{
  "errs": [],
  "args": {
    "_": ["--help"]
  }
}
```

The error is gone and we have `args`.

Note that `git` does not have options.
This is why `"--help"` does not mean anything to the `parser`.
It ends up in the <em>rest array</em> `_`, that holds all tokens that could not be interpreted.

</details>

### Options

Let us add a `--help` option to the `git` `command`, next:

```js
// ...
const {flag} = require('shargs/opts')

const opts = [
  flag('help', ['--help'])
]

const git = command('git', opts)
// ...
```

We chose to make `help` a `flag`, so we can just call `--help` without any values.

Now we should be able to parse `--help`:

<details>
<summary>
<code>./git --help</code>
</summary>

<br />

```json
{
  "errs": [],
  "args": {
    "_": [],
    "help": { "type": "flag", "count": 1 }
  }
}
```

The `_` array in `args` is empty, and we have successfully parsed `--help` into the `help` field.

Note that while `--help` is the command-line argument we use, `help` (without `--`) is the field name.
This is reflected in the definition of the `flag`.
Shargs separates between the external API of providing an argument, and the internal API of storing the values.

</details>

### Parser Stages

`--help` is parsed into a weird flag object with a `count` field.
However, we do not need to know, how often `--help` has been called,
we only need to know if it has been provided at least once.

So we do not need a `flag` value, a `bool` is enough:

```js
// ...
const {flagAsBool} = require('shargs/parser')
// ...
const stages = {
  args: [flagAsBool('help')]
}

const parser = parserSync(stages)
//...
```

We have modified `parser`'s behavior by adding an `args` stage to its `stages` parameter.
[`flagAsBool`](https://github.com/Yord/shargs/tree/0.24.4#flagAsBool) transforms a `flag` with a given key (here `'help'`) into a bool.

Shargs works in seven different [steps](https://github.com/Yord/shargs/tree/0.24.4#stages) that each take one or more stages.
The `args` step is at the sixth position.

Let us see `stages` in action.

<details>
<summary>
<code>./git --help</code>
</summary>

<br />

```json
{
  "errs": [],
  "args": {
    "_": [],
    "help": true
  }
}
```

The `help` field is now `true`.

</details>

### Subcommands

Git has many subcommands that do various tasks.
Let us start by adding the `init` `subcommand`:

```js
// ...
const {flag, subcommand} = require('shargs/opts')
// ...
const init = subcommand([])

const opts = [
  // ...
  init('init', ['init'])
]
// ...
```

For now, `init` is a subcommand without any options.
Note, that we do not call it as `--init`, like we did with `--help`, but just as `init`.

Shargs does not force you to use a specific syntax for argument names.
You can choose any string you like, as long as it does not contain whitespaces.
Since, by convention, subcommands do not start with `--`, I decided for this case, to just use `init` as an argument.

Let us see what calling `init` yields.

<details>
<summary>
<code>./git init</code>
</summary>

<br />

```json
{
  "errs": [],
  "args": {
    "_": [],
    "init": {
      "_": []
    }
  }
}
```

`args` now has an `init` field.
Note, that although we did not provide any other arguments and although `init` has no options,
the `init` field has a `_` array in `args`.
This is because every `command` and `subcommand` may receive arguments it does not recognize
and thus has its own rest array `_`.

</details>

### Subcommand Options

Like `command`s, `subcommand`s may have options of their own:

```js
// ...
const init = subcommand([
  flag('quiet', ['-q', '--quiet'])
])
// ...
```

We have added a `quiet` flag, that is meant to suppress unnecessary output.

Let us see if it works with `init`.

<details>
<summary>
<code>./git init -q</code>
</summary>

<br />

```json
{
  "errs": [],
  "args": {
    "_": [],
    "init": {
      "_": [],
      "quiet": {
        "type": "flag",
        "count": 1
      }
    }
  }
}
```

Indeed, the `init` field has now a nested `quiet` field with a `flag` value.

</details>

### Short Option Groups

Many commands count their `quiet` arguments and hide logging information based on how often `-q` was passed.
Usually, these commands let you write `-qqq` instead of `-q -q -q`.
A feature shargs calls <em>short option groups</em>.
They are supported by adding the [`splitShortOpts`](https://github.com/Yord/shargs/tree/0.24.4#splitShortOpts) stage:

```js
// ...
const {flagAsBool, splitShortOpts} = require('shargs/parser')

const stages = {
  argv: [splitShortOpts],
  args: [flagAsBool('help')]
}
// ...
```

Now we should be able to define how quiet the output should be.

<details>
<summary>
<code>./git init -qqq</code>
</summary>

<br />

```json
{
  "errs": [],
  "args": {
    "_": [],
    "init": {
      "_": [],
      "quiet": {
        "type": "flag",
        "count": 3
      }
    }
  }
}
```

`splitShortOpts` actually rewrites `./git init -qqq` to `./git init -q -q -q` internally.
Note that the `quiet` field `count` is `3`, which is exactly the number of `-q` arguments.

</details>

### Transforming Flags

Like `help`, we do not need to know in the results, that `quiet` was a `flag`.
Storing its `count` as a number suffices:

```js
// ...
const {flagsAsBools, flagAsNumber, splitShortOpts} = require('shargs/parser')

const stages = {
  argv: [splitShortOpts],
  args: [flagAsNumber('quiet'), flagsAsBools]
}
// ...
```

We have changed two things, here:
First, we have added the [`flagAsNumber`](https://github.com/Yord/shargs/tree/0.24.4#flagAsNumber) stage to transform `quiet`.
Second, we have generalized `flagAsBool` to [`flagsAsBools`](https://github.com/Yord/shargs/tree/0.24.4#flagsAsBools),
a stage that transforms all remaining `flag`s to bools.

<details>
<summary>
<code>./git --help init -qqq</code>
</summary>

<br />

```json
{
  "errs": [],
  "args": {
    "_": [],
    "help": true,
    "init": {
      "_": [],
      "quiet": 3
    }
  }
}
```

The `help` field is still a bool, while the `quiet` field is just a number, now.

</details>

### Rest Arrays

We have already seen that rest arrays exist, now, let us get a feeling for how they work:

<details>
<summary>
<code>./git --help init -qqq commit</code>
</summary>

<br />

```json
{
  "errs": [],
  "args": {
    "_": [],
    "help": true,
    "init": {
      "_": ["commit"],
      "quiet": 3
    }
  }
}
```

</details>

<details>
<summary>
<code>./git init -qqq --help commit</code>
</summary>

<br />

```json
{
  "errs": [],
  "args": {
    "_": ["commit"],
    "help": true,
    "init": {
      "_": [],
      "quiet": 3
    }
  }
}
```

</details>

<br />

In the first case, `"commit"` is still considered a part of the `init` `subcommand`.
In the second case, `"commit"` is stored in the `git` `command`'s rest array.

The reason for this difference is `--help`.
Upon reaching `--help`, the `parser` tries to find the token in its options.
In the second case, the parser first looks for `--help` in `init`'s options.
Since it does not find an option with the [`args`](https://github.com/Yord/shargs/tree/0.24.4#args) `--help`,
it continues searching in `init`'s parent `git`.
Here, it finds the `help` option that has a `--help` argument.

However, it has left the `init` `subcommand`'s scope for good and is now back in `git`'s scope.
This is why `commit` is in `git`'s rest array, while it is still in `init`'s rest array in the first case.

### Positional Arguments

Besides arguments that are passed by argument name (aka options), many commands have arguments that are passed by position.
Shargs supports both, options and positional arguments:

```js
// ...
const {flag, string, subcommand, variadicPos} = require('shargs/opts')

const commit = subcommand([
  flag('all', ['-a', '--all']),
  string('message', ['-m', '--message']),
  variadicPos('file')
])

const opts   = [
  // ...
  commit('commit', ['commit'])
]
// ...
```

We have added the new `commit` `subcommand` that has three different kinds of arguments:

1.  `all` is a `flag` that has just an argument name (`-a` or `--all`), but no argument values.
2.  `message` is a `string` option that has an argument name (`-m` or `--message`)
    as well as an argument value (one `string`).
3.  `file` is a positional argument that has no argument name, only an argument value.
    `file` is also [variadic](https://github.com/Yord/shargs/tree/0.24.4#variadic-pos-arg),
    meaning it takes not one, but any number of values.

Let us test `commit`.

<details>
<summary>
<code>./git commit -a -m 'First commit' package.json README.md</code>
</summary>

<br />

```json
{
  "errs": [],
  "args": {
    "_": [],
    "commit": {
      "_": [],
      "all": true,
      "message": "First commit",
      "file": [
        "package.json",
        "README.md"
      ]
    }
  }
}
```

The `message` field is indeed a string and `file` collects any number of argument values.

</details>

### Multiple Subcommands

One thing that makes shargs special,
is its support for specifying [multiple subcommands](https://github.com/Yord/shargs/tree/0.24.4#multiple-subcommands).

Imagine you could write the following in real `git`:

<details>
<summary>
<code>./git --help init -qqq commit -a -m 'First commit' package.json README.md</code>
</summary>

<br />

```json
{
  "errs": [],
  "args": {
    "_": [],
    "help": true,
    "init": {
      "_": [],
      "quiet": 3
    },
    "commit": {
      "_": [],
      "all": true,
      "message": "First commit",
      "file": [
        "package.json",
        "README.md"
      ]
    }
  }
}
```

Note that shargs does not necessarily retain the `subcommand`'s order.
Since JavaScript objects' order is not specified and depends on the engine,
you should not rely on the object's field order.

</details>

### Usage Documentation

Each command has to have a help page.
We can automatically generate a help text based on `git` with
[`shargs/usage`](https://github.com/Yord/shargs/tree/0.24.4#automatic-usage-documentation-generation):

```js
// ...
const {optsList} = require('shargs/usage')
// ...

if (res.args.help) {
  const help = optsList(git)()
  console.log(help)
} else {
  console.log(JSON.stringify(res, null, 2))
}
```

If the `args` field of the `parse` results has a truthy `help` field, we `log` `help` to the `console`.
For now, `help` is just an `optsList` that layouts `git`'s options.

<details>
<summary>
<code>./git --help</code>
</summary>

<br />

```bash
--help                                                                          
init                                                                            
commit                                                                          
```

We get a plain list of all `git` options.
Note, that the list is not sorted, but is presented in the order we have specified in `opts`.

</details>

### Descriptions

We can enhance the usage documentation by adding [`desc`](https://github.com/Yord/shargs/tree/0.24.4#desc)riptions
to `git`'s options:

```js
// ...
const opts = [
  flag('help', ['--help'], {desc: 'Print this help message.'}),
  init('init', ['init'], {desc: 'Create an empty Git repository or reinitialize an existing one.'}),
  commit('commit', ['commit'], {desc: 'Record changes to the repository.'})
]
// ...
```

The `--help` page is more helpful, now.

<details>
<summary>
<code>./git --help</code>
</summary>

<br />

```bash
--help                   Print this help message.                               
init                     Create an empty Git repository or reinitialize an      
                         existing one.                                          
commit                   Record changes to the repository.                      
```

</details>

### Subcommands Documentation

`--help` would be even more helpful, if it would also show the `subcommand`s' options:

```js
// ...
const {optsLists} = require('shargs/usage')
// ...
const init = subcommand([
  flag('quiet', ['-q', '--quiet'], {desc: 'Only print error and warning messages.'})
])

const commit = subcommand([
  flag('all', ['-a', '--all'], {desc: 'Automatically stage files that have been modified and deleted.'}),
  string('message', ['-m', '--message'], {desc: 'Use <string> as the commit message.'}),
  variadicPos('file', {desc: 'A list of files to commit.'})
])
// ...
const git = command('git', opts, {desc: 'A simple command-line interface for git.'})
// ...
  const help = optsLists(git)()
// ...
```

We have added `desc`riptions to all options, now.

Another update is easy to miss:
Instead of `optsList`, we use `optsLists` (plural) to generate our usage documentation.
`optsLists` recursively documents all `subcommand` options.

<details>
<summary>
<code>./git --help</code>
</summary>

<br />

```bash
--help                   Print this help message.                               
init                     Create an empty Git repository or reinitialize an      
                         existing one.                                          
    -q, --quiet          Only print error and warning messages.                 
commit                   Record changes to the repository.                      
    -a, --all            Automatically stage files that have been modified and  
                         deleted.                                               
    -m,                  Use <string> as the commit message.                    
    --message=<string>                                                          
    <file>...            A list of files to commit.                             
```

All options are now documented.

</details>

### Argument Descriptions

Have you recognized the line break, because `--message=<string>` is just a little bit too long?
Let us fix that:

```js
// ...
  string('message', ['-m', '--message'], {descArg: 'msg', desc: 'Use <msg> as the commit message.'}),
// ...
```

We have used the [`descArg`](https://github.com/Yord/shargs/tree/0.24.4#descArg) field to change `message`'s value description
from `<string>` to `<msg>`.
This has given us just enough space to fix the layout.

<details>
<summary>
<code>./git --help</code>
</summary>

<br />

```bash
--help                   Print this help message.                               
init                     Create an empty Git repository or reinitialize an      
                         existing one.                                          
    -q, --quiet          Only print error and warning messages.                 
commit                   Record changes to the repository.                      
    -a, --all            Automatically stage files that have been modified and  
                         deleted.                                               
    -m, --message=<msg>  Use <msg> as the commit message.                       
    <file>...            A list of files to commit.                             
```

</details>

### Enhance Help

We can do better than just displaying the options in a list and add some more elements to our help:

```js
// ...
const {desc, optsLists, space, synopses, usage} = require('shargs/usage')
// ...
if (res.args.help) {
  const help = usage([
    synopses,
    space,
    optsLists,
    space,
    desc
  ])(git)()
  console.log(help)
} else {
  console.log(JSON.stringify(res, null, 2))
}
```

We have also added `synopses` (mind the plural) and `git`'s `desc`ription to `help`.
`usage` is used to group these elements together, so we only have to pass the `git` `command` once.

<details>
<summary>
<code>./git --help</code>
</summary>

<br />

```bash
git [--help]                                                                    
git init [-q|--quiet]                                                           
git commit [-a|--all] [-m|--message] [<file>...]                                
                                                                                
--help                   Print this help message.                               
init                     Create an empty Git repository or reinitialize an      
                         existing one.                                          
    -q, --quiet          Only print error and warning messages.                 
commit                   Record changes to the repository.                      
    -a, --all            Automatically stage files that have been modified and  
                         deleted.                                               
    -m, --message=<msg>  Use <msg> as the commit message.                       
    <file>...            A list of files to commit.                             
                                                                                
A simple command-line interface for git.                                        
```

The usage documentation comes together nicely!

</details>

### With Style

Now that the contents of our usage documentation are complete, let us finish off by adding some styles:

```js
// ...
const {desc, optsListsWith, space, synopses, usage} = require('shargs/usage')
// ...
  const style = {
    line: [{width: 50}],
    cols: [{width: 20}, {width: 30}]
  }
  const help = usage([
    synopses,
    space,
    optsListsWith({pad: 2}),
    space,
    desc
  ])(git)(style)
// ...
```

First off, we change the left padding of the `optsLists` to be just `2`
with [`optsListsWith`](https://github.com/Yord/shargs/tree/0.24.4#optsListsWith).
Then, we add a `style` object and pass it to `usage`.

The `style` says that `line`s should be `50` columns wide,
while `cols` specifies the first column's `width` to be `20` and the second's to be `30`.

You can read up in shargs' documentation (e.g. with [`optsListsWith`](https://github.com/Yord/shargs/tree/0.24.4#optsListsWith)),
what `style` field a component uses for its layout.
You could also invent new `style` fields and configure components to use those.

Let us print `help` one last time.

<details>
<summary>
<code>./git --help</code>
</summary>

<br />

```bash
git [--help]                                      
git init [-q|--quiet]                             
git commit [-a|--all] [-m|--message] [<file>...]  
                                                  
--help              Print this help message.      
init                Create an empty Git repository
                    or reinitialize an existing   
                    one.                          
  -q, --quiet       Only print error and warning  
                    messages.                     
commit              Record changes to the         
                    repository.                   
  -a, --all         Automatically stage files that
                    have been modified and        
                    deleted.                      
  -m,               Use <msg> as the commit       
  --message=<msg>   message.                      
  <file>...         A list of files to commit.    
                                                  
A simple command-line interface for git.          
```

The usage documentation is now very compact.
If you do not like it, yet, keep changing `style` until you are satisfied.

</details>

## Reporting Issues

Please report issues [in the `shargs` tracker][issues]!

## License

`shargs-tutorial-git` is [MIT licensed][license].



[contribute]: https://github.com/Yord/shargs#contributing
[issues]: https://github.com/Yord/shargs/issues
[license]: https://github.com/Yord/shargs-tutorial-git/blob/master/LICENSE
[node]: https://nodejs.org/
[repl]: https://github.com/Yord/shargs-tutorial-git/blob/master/repl
[shargs]: https://github.com/Yord/shargs
[shield-license]: https://img.shields.io/badge/license-MIT-yellow.svg?labelColor=313A42
[shield-node]: https://img.shields.io/node/v/shargs?color=red&labelColor=313A42
[shield-prs]: https://img.shields.io/badge/PRs-welcome-green.svg?labelColor=313A42
[teaser]: https://github.com/Yord/shargs-tutorial-git/blob/master/teaser.gif?raw=true