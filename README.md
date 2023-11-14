[![Build Status][build]][build-url]
[![npm][npm]][npm-url]
[![node][node]][node-url]

<div align="center">
  <a href="https://github.com/webpack/webpack">
    <img width="200" height="200" alt="Webpack" src="https://webpack.js.org/assets/icon-square-big.svg" />
  </a>

  <img height="200" src="./docs/wasm-gopher.png" alt="Go Gopher with WASM logo" />

  <h1>Golang WebAssembly Async Loader</h1>
  <p>Generates a WASM package from Golang and provides an async interface for working with it</p>
  <p>@fiedka/golang-wasm-async-loader is a fork of golang-wasm-async-loader updated for Golang up to v1.17</p>
</div>

<h2 align="center">Install</h2>

```bash
npm install --save-dev @versori/golang-wasm-async-loader
```

This is a loader for [webpack](https://webpack.js.org/) that is used for generating [WebAssembly](https://webassembly.org/) (aka WASM) bundles from [Go](https://golang.org).

The JavaScript bridge that is then generated for webpack will expose the WebAssembly functions as a Promise for interacting with.

## webpack config

```js
module.exports = {
    ...
    module: {
        rules: [
            {
                test: /\.go/,
                use: ['golang-wasm-async-loader']
            }
        ]
    },
    node: {
        fs: 'empty'
    }
};
```

# Using in your code

You import your Go code just like any other JavaScript module you might be working with. The webpack loader will export a default export that has the functions you registered in Go on it. Unfortunately it currently doesn't provide autocomplete of those function names as they are runtime defined.

```js
import wasm from './main.go'

async function init() {
  const result = await wasm.add(1, 2);
  console.log(result);

  const someValue = await wasm.someValue();
  console.log(someValue);
}
```

Here's the `main.go` file:

```go
package main

import (
  "strconv"
  "syscall/js"
  "github.com/fiedka/webpack-golang-wasm-async-loader/gobridge"
)

func add(i []js.Value) (interface{},error) {
 ret := 0

 for _, item := range i {
  val, _ := strconv.Atoi(item.String())
  ret += val
 }

 return ret, nil
}

func main() {
 c := make(chan struct{}, 0)

 gobridge.RegisterCallback("add", add)
 gobridge.RegisterValue("someValue", "Hello World")

 <-c
}
```

## How does it work?

As part of this repository a Go package has been created to improve the interop between the Go WASM runtime and work with the async pattern the loader defines.

To do this a function is exported from the package called `RegisterCallback` which takes two arguments:

* A `string` representing the name to register it as in JavaScript (and what you'll call it using)
* The `func` to register as a callback
  * The `func` must has a signature of `(args js.Value) (interface{}, error)` so you can raise an error if you need

If you want to register a static value that's been created from Go to be available in JavaScript you can do that with `RegisterValue`, which takes a name and a value. Values are converted to functions that return a Promise so they can be treated asynchronously like function invocations.

In JavaScript a global object is registered as `__gobridge__` which the registrations happen against.

## Examples

Examples are provided for a CLI using NodeJS and for web using either React or Svelte. These are in the [`examples`](https://github.com/fiedka/webpack-golang-wasm-async-loader/tree/main/examples) directory, each with its own development and build environment.

To make an example stand-alone, copy of the corresponding example to a new directory (outside the plugin directory) and then modify the example's `webpack.config.js` so that the `.go` loader refers to this plugin. Then add it to the example's dependencies as follows.

For use with **Go 1.13**, use happybeing's v1.0.6 of the plugin:

```bash
npm add --save-dev golang-wasm-async-loader2@1.0.6`
```

For use with **Go 1.15**, use happybeing's v1.1.0 of the plugin:

```bash
npm add --save-dev golang-wasm-async-loader2@1.1.0`
```

For use with **Go 1.16**, use happybeing's v3.0.2 of the plugin:

```bash
npm add --save-dev @fiedka/golang-wasm-async-loader@3.0.2`
```

## Publishing

This is configured to publish to @versori/golang-wasm-async-loader hosted by Versori.
To publish a new version update `package.json` and run the following commands

```bash
npm run ar-login
npm install
npm run build
npm pack
npm publish
```

## Environment

To build your project (and the examples) you will need the GOROOT environment variable set. So for example if using the bash shell:

node example:

```bash
GOROOT=`go env GOROOT` npm run predemo
npm run demo
```

web example (with hot reloading):

```bash
GOROOT=`go env GOROOT` npm run build
GOROOT=`go env GOROOT` npm run start
```

# Licence

MIT

# Credit

Aaron Powell (updates up to Go 1.15 by Mark Hughes)

[build]: https://img.shields.io/github/actions/workflow/status/fiedka/webpack-golang-wasm-async-loader/buid.yml
[build-url]: https://github.com/fiedka/webpack-golang-wasm-async-loader/actions

[npm]: https://img.shields.io/npm/v/@fiedka/golang-wasm-async-loader.svg
[npm-url]: https://npmjs.com/package/@fiedka/golang-wasm-async-loader

[node]: https://img.shields.io/node/v/@fiedka/golang-wasm-async-loader.svg
[node-url]: https://nodejs.org
