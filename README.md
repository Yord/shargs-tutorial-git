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