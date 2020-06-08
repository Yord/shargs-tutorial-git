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
$ npm install --save --save-exact shargs@0.24.4 shargs-opts@0.5.0 shargs-parser@0.6.0 shargs-usage@0.6.0
```

Note that we install fixed versions.
This tutorial should work with these exact versions, and might work with newer versions.
However, this tutorial may not work if you use different versions.

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
  errs: [
    {
      code: "CommandExpected",
      msg: "Expected a command with a string 'key' field and an 'opts' array.",
      info: {opt: {}}
    }
  ],
  args: {
    _: []
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

## Reporting Issues

Please report issues [in the `shargs` tracker][issues]!

## License

`shargs-example-repl` is [MIT licensed][license].



[contribute]: https://github.com/Yord/shargs#contributing
[issues]: https://github.com/Yord/shargs/issues
[license]: https://github.com/Yord/shargs-example-repl/blob/master/LICENSE
[node]: https://nodejs.org/
[repl]: https://github.com/Yord/shargs-example-repl/blob/master/repl
[shargs]: https://github.com/Yord/shargs
[shield-license]: https://img.shields.io/badge/license-MIT-yellow.svg?labelColor=313A42
[shield-node]: https://img.shields.io/node/v/shargs?color=red&labelColor=313A42
[shield-prs]: https://img.shields.io/badge/PRs-welcome-green.svg?labelColor=313A42
[teaser]: https://github.com/Yord/shargs-tutorial-git/blob/master/teaser.gif?raw=true