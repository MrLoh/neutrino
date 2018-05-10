# Library

`@neutrinojs/library` is a Neutrino preset that supports creating JS libraries for Web or Node.js targets.

![](https://img.shields.io/npm/v/@neutrinojs/library.svg) ![](https://img.shields.io/npm/dt/@neutrinojs/library.svg)

## Features

* Zero upfront configuration necessary to start developing and building a JavaScript library
* Modern Babel compilation supporting ES modules, async functions, and dynamic imports
* Defaults to UMD-based output for consumption in a variety of environments
* Supports automatically-wired sourcemaps
* Tree-shaking to create smaller bundles
* Automatically marks dependencies as external
* Easily extensible to customize your library as needed

## Requirements

* Node.js v8.3+
* Yarn v1.2.1+, or npm v5.4+
* Neutrino v8

## Installation

`@neutrinojs/library` can be installed via the Yarn or npm clients. Inside your project, make sure `neutrino` and `@neutrinojs/library` are development dependencies.

#### Yarn

```bash
❯ yarn add --dev neutrino @neutrinojs/library
```

#### npm

```bash
❯ npm install --save-dev neutrino @neutrinojs/library
```

If you want to have automatically wired sourcemaps added to your project, add `source-map-support`:

#### Yarn

```bash
❯ yarn add source-map-support
```

#### npm

```bash
❯ npm install --save source-map-support
```

## Project Layout

`@neutrinojs/library` follows the standard [project layout](https://github.com/mozilla-neutrino/neutrino-dev/tree/e79fa15034476cb1263299616b64545268e6f570/docs/project-layout/README.md) specified by Neutrino. This means that by default all library source code should live in a directory named `src` in the root of the project. This includes JavaScript files that would be available to your compiled project.

## Quickstart

The fastest way to get started is by using the `create-project` scaffolding tool. Don’t want to use the CLI helper? No worries, we have you covered with the [manual installation](library.md#manual-installation).

### create-project

Run the following command to start the process. Substitute `<directory-name>` with the directory name you wish to create for this project.

#### Yarn

```text
❯ yarn create @neutrinojs/project <directory-name>
```

_Note: The _`create`_ command is a shorthand that helps you do two things at once. See the _[_Yarn create docs_](https://yarnpkg.com/lang/en/docs/cli/create)_ for more details._

#### npm/npx

[`npx`](https://github.com/zkat/npx) comes pre-installed with `npm`. If you’re running an older version of `npm`, then `npm install -g npm` to update to the latest version.

```text
❯ npx @neutrinojs/create-project <directory-name>
```

The CLI helper will prompt for the project to scaffold, and will offer to set up a test runner as well as linting to your project. Refer to the [Create new project](../installation/create-new-project.md) section for details on all available options.

### Manual Installation

After installing Neutrino and the Library preset, add a new directory named `src` in the root of the project, with a single JS file named `index.js` in it.

```bash
❯ mkdir src && touch src/index.js
```

Edit your `src/index.js` file with the following:

```javascript
// Create a simple logger library that only logs when
// debug mode is enabled, and logs to the appropriate level
export default class Logger {
  constructor(level, debug = true) {
    this.level = level;
    this.debug = debug;
  }

  log(...args) {
    if (this.debug) {
      console[level](...args);
    }
  }
}
```

Now edit your project's package.json to add commands for building the library.

```javascript
{
  "name": "super-logger",
  "scripts": {
    "build": "neutrino build"
  }
}
```

When using this library, you **must** use a `.neutrinorc.js` file in your project to specify the library name. Add this preset to your `use` array and define the `name` option:

```javascript
module.exports = {
  use: [
    ['@neutrinojs/library', { name: 'Logger' }]
  ]
};
```

You can now build your library!

#### Yarn

```bash
❯ yarn build
```

#### npm

```bash
❯ npm start
```

## Building

`@neutrinojs/library` builds assets to the `build` directory by default when running `neutrino build`. Using the quick start example above as a reference:

```bash
❯ yarn build

Build completed in 0.419s

Hash: 89e4fb250fc535920ba4
Version: webpack 3.8.1
Time: 424ms
       Asset     Size  Chunks             Chunk Names
    Logger.js  4.29 kB       0  [emitted]  index
Logger.js.map  3.73 kB       0  [emitted]  index
✨  Done in 1.51s.
```

You should specify a `main` property in your package.json pointing to your primary built main entry point. Also when publishing your project to npm, consider excluding your `src` directory by using the `files` property to whitelist `build`, or via `.npmignore` to blacklist `src`.

```javascript
{
  "name": "super-logger",
  "main": "build/index.js",
  "files": [
    "build"
  ]
}
```

Your built library can now be consumed with ES imports, CJS require, AMD require, or even script tags:

```javascript
// ES imports
import Logger from 'super-logger';

// CJS require
const Logger = require('super-logger');

// AMD require
require(['super-logger'], (Logger) => {
  // ...
});
```

```markup
<!-- script tags -->
<script src="/path/to/super-logger"></script>

<!--
  once published to npm, you can even use a script tag
  to point to unpkg
-->
<script src="https://unpkg.com/super-logger"></script>
```

## Preset options

You can provide custom options and have them merged with this preset's default options to easily affect how this preset builds. You can modify Library preset settings from `.neutrinorc.js` by overriding with an options object. Use an array pair instead of a string to supply these options in `.neutrinorc.js`.

The following shows how you can pass an options object to the Library preset and override its options, showing the defaults where applicable:

```javascript
module.exports = {
  use: [
    ['@neutrinojs/library', {
      // REQUIRED. Sets the name of the library, along with its
      // output filename.
      name: '<YOUR_LIBRARY_NAME>',

      // Compile the library for use in a specific environment.
      // Defaults to 'web', but can be switched to 'node' as well
      target: 'web',

      // Configure how the library will be exposed. Keeping this set
      // to 'umd' ensures compatibility with a large number of module
      // systems, but you can override if you want to produce a smaller
      // bundle targeted to specific module systems like 'commonjs2' (CJS).
      libraryTarget: 'umd',

      // Override options passed to webpack-node-externals,
      // such as whitelisting dependencies for bundling.
      externals: {},

      // Remove the contents of the output directory prior to building.
      // Set to false to disable cleaning this directory
      clean: {
        paths: [neutrino.options.output]
      },

      // Add additional Babel plugins, presets, or env options
      babel: {
        // Override options for @babel/preset-env
        presets: [
          ['@babel/preset-env', {
            // Passing in browser targets to @babel/preset-env will replace them
            // instead of merging them when using the 'web' target
            targets: {
              browsers: [
                'last 1 Chrome versions',
                'last 1 Firefox versions'
              ]
            }
          }]
        ]
      }
    }]
  ]
};
```

_Example: Override the library Babel compilation target to Node.js v6 and commonjs2 module:_

```javascript
module.exports = {
  use: [
    ['@neutrinojs/library', {
      name: 'Logger',
      target: 'node',
      libraryTarget: 'commonjs2',
      // Add additional Babel plugins, presets, or env options
      babel: {
        // Override options for @babel/preset-env
        presets: [
          ['@babel/preset-env', {
            targets: {
              node: '6.0'
            }
          }]
        ]
      }
    }]
  ]
};
```

## Customizing

To override the build configuration, start with the documentation on [customization](https://neutrino.js.org/customization). `@neutrinojs/library` creates some conventions to make overriding the configuration easier once you are ready to make changes.

By default Neutrino, and therefore this preset, creates a single **main** `index` entry point to your library, and this maps to the `index.*` file in the `src` directory. This means that this preset is optimized toward a single main entry to your library. Code not imported in the hierarchy of the `index` entry will not be output to the bundle. To overcome this you must either define more mains via [`options.mains`](https://neutrino.js.org/customization#optionsmains), import the code path somewhere along the `index` hierarchy, or define multiple configurations in your `.neutrinorc.js`.

### External dependencies

This preset automatically marks all dependencies as external to your library, meaning that any dependencies you import **will not be bundled** with your library. This helps prevent your library from bloating, but means users of your library will be installing or using your dependencies as defined in package.json. You can override this in the library preset options by passing further options to the `externals` property. This accepts an options object format defined by [webpack-node-externals](https://www.npmjs.com/package/webpack-node-externals#configuration), to which you can provide a `whitelist` value.

_The whitelist will override which dependencies are **bundled** in your library. Any dependency not matched by this whitelist is considered a peer of your library, and will **not be bundled**._

_Example: The following example library _`redux-example`_ has the following package.json, marking _`redux`_ and _`mitt`_ as dependencies, but only lists _`mitt`_ in whitelist. This bundles _`mitt`_ along with the library, so the library consumer does not need to explicitly import/require/script it prior. The _`redux`_ dependency is not in the whitelist, so it will not be bundled, and is considered a peer of _`redux-example`_._

```javascript
{
  "name": "redux-example",
  "version": "1.0.0",
  "main": "build/reduxExample.js",
  "scripts": {
    "build": "neutrino build"
  },
  "dependencies": {
    "mitt": "*",
    "redux": "*"
  },
  "devDependencies": {
    "neutrino": "*",
    "@neutrinojs/library": "*"
  }
}
```

```javascript
module.exports = {
  use: [
    ['@neutrinojs/library', {
      name: 'reduxExample',
      externals: {
        whitelist: ['mitt']
      }
    }]
  ]
};
```

```javascript
// ES imports
import { createStore } from 'redux';
import reduxExample from 'redux-example';

// CJS require
const { createStore } = require('redux');
const reduxExample = require('redux-example');

// AMD require
require(['redux', 'redux-example'], ({ createStore }, reduxExample) => {
  // ...
});
```

```markup
<!-- script tags -->
<script src="/path/to/redux"></script>
<script src="/path/to/redux-example"></script>
<script>
  const { createStore } = window.redux;
  window.reduxExample // ...
</script>

<!--
  once published to npm, you can even use a script tag
  to point to unpkg
-->
<script src="https://unpkg.com/redux"></script>
<script src="https://unpkg.com/redux-example"></script>
<script>
  const { createStore } = window.redux;
  window.reduxExample // ...
</script>
```

## Generating multiple builds

The `@neutrinojs/library` middleware can be used in conjunction with the [`@neutrinojs/fork` middleware](https://neutrinojs.org/packages/fork) to generate multiple library outputs when building. Follow the instructions to install the fork middleware, and change your `.neutrinorc.js` format as follows:

```javascript
const name = 'Logger';

module.exports = {
  use: [
    (neutrino) => {
      neutrino.on('prebuild', () => neutrino.use('@neutrinojs/clean'));
    },
    ['@neutrinojs/fork', {
      configs: {
        // Create a named entry for each build type.
        // You will most likely want to disable cleaning
        // the output directory until prior to building
        umd: ['@neutrinojs/library', { name, clean: false }],
        commonjs2: ['@neutrinojs/library', { name, clean: false }]
      }
    }]
  ]
};
```

### Rules

The following is a list of rules and their identifiers which can be overridden:

| Name | Description | Environments and Commands |
| --- | --- | --- |
| `compile` | Compiles JS files from the `src` directory using Babel. Contains a single loader named `babel` | all |
| `worker` | Allows importing Web Workers automatically with `.worker.*` extensions. Contains a single loader named `worker`. | all |

### Plugins

The following is a list of plugins and their identifiers which can be overridden:

_Note: Some plugins are only available in certain environments. To override them, they should be modified conditionally._

| Name | Description | Environments and Commands |
| --- | --- | --- |
| `banner` | Injects source-map-support into the main entry points of your application if detected in `dependencies` or `devDependencies` of your package.json. | Only when `source-map-support` is installed |
| `clean` | Clears the contents of `build` prior to creating a production bundle. | `build` command |

### Override configuration

By following the [customization guide](../customization.md) and knowing the rule, loader, and plugin IDs above, you can override and augment the build by by providing a function to your `.neutrinorc.js` use array. You can also make these changes from the Neutrino API in custom middleware.

_Example: Allow importing modules with a _`.esm`_ extension._

```javascript
module.exports = {
  use: [
    ['@neutrinojs/library', { /* ... */ }],
    (neutrino) => {
      neutrino.config.resolve.extensions.add('.esm')
    }
  ]
};
```

## Contributing

This preset is part of the [neutrino-dev](https://github.com/mozilla-neutrino/neutrino-dev) repository, a monorepo containing all resources for developing Neutrino and its core presets and middleware. Follow the [contributing guide](../contributing/) for details.
